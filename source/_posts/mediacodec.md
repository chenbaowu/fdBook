---
title: mediacodec 视频硬编解码
date: 2018-03-11 14:44:42
categories: android
tags: 编解码
---

## mediacodec 从零开始编解码

首先先了解一下安卓为我们提供的api
``` bash
MediaCodec用于创建视音频编解码器，通过它可以对视音频数据进行编解码操作，它是编解码功能的核心类。

MediaExtractor相当于一个reader，它用于读取媒体文件，并提取出其中的视音频数据。

MediaMuxer相当于一个writer，它用于将内存中的视音频数据写到文件中。

MediaFormat即媒体格式类，它用于描述媒体的格式参数，如视频帧率、音频采样率等

 mBufferInfo = new MediaCodec.BufferInfo(); // 用于描述编码得到的byte[]数据的相关信息
```

<!-- more -->

![](http://img.blog.csdn.net/20140528102358468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW91c2VfMTg5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 解码

从代码解析流程

#### 初始化

```bash
/**
     * 初始化解码器
     */
    private void initMediaDecode() {
        try {
            mediaExtractor=new MediaExtractor();//此类可分离视频文件的音轨和视频轨道
            mediaExtractor.setDataSource(srcPath);//媒体文件的位置
            for (int i = 0; i < mediaExtractor.getTrackCount(); i++) {//遍历媒体轨道 此处我们传入的是音频文件，所以也就只有一条轨道
                MediaFormat format = mediaExtractor.getTrackFormat(i);
                String mime = format.getString(MediaFormat.KEY_MIME);
                if (mime.startsWith("audio/")) {//获取音频轨道
//                    format.setInteger(MediaFormat.KEY_MAX_INPUT_SIZE, 200 * 1024);
                    mediaExtractor.selectTrack(i);//选择此音频轨道
                    mediaDecode = MediaCodec.createDecoderByType(mime);//创建Decode解码器
                    mediaDecode.configure(format, null, null, 0);
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        if (mediaDecode == null) {
            Log.e(TAG, "create mediaDecode failed");
            return;
        }
        mediaDecode.start();//启动MediaCodec ，等待传入数据
        decodeInputBuffers=mediaDecode.getInputBuffers();//MediaCodec在此ByteBuffer[]中获取输入数据
        decodeOutputBuffers=mediaDecode.getOutputBuffers();//MediaCodec将解码后的数据放到此ByteBuffer[]中 我们可以直接在这里面得到PCM数据
        decodeBufferInfo=new MediaCodec.BufferInfo();//用于描述解码得到的byte[]数据的相关信息
        showLog("buffers:" + decodeInputBuffers.length);
    }
```

### 解码过程

```bash
 private void srcAudioFormatToPCM() {
        for (int i = 0; i < decodeInputBuffers.length-1; i++) {
            int inputIndex = mediaDecode.dequeueInputBuffer(-1);//获取可用的inputBuffer -1代表一直等待，0表示不等待 建议-1,避免丢帧
            if (inputIndex < 0) {
                codeOver =true;
                return;
            }

            ByteBuffer inputBuffer = decodeInputBuffers[inputIndex];//拿到inputBuffer
            inputBuffer.clear();//清空之前传入inputBuffer内的数据
            int sampleSize = mediaExtractor.readSampleData(inputBuffer, 0);//MediaExtractor读取数据到inputBuffer中
            if (sampleSize <0) {//小于0 代表所有数据已读取完成
                codeOver=true;
            }else {
                mediaDecode.queueInputBuffer(inputIndex, 0, sampleSize, 0, 0);//通知MediaDecode解码刚刚传入的数据
                mediaExtractor.advance();//MediaExtractor移动到下一取样处
                decodeSize+=sampleSize;
            }
        }

        //获取解码得到的byte[]数据 参数BufferInfo上面已介绍 10000同样为等待时间 同上-1代表一直等待，0代表不等待。此处单位为微秒
        //此处建议不要填-1 有些时候并没有数据输出，那么他就会一直卡在这 等待
        int outputIndex = mediaDecode.dequeueOutputBuffer(decodeBufferInfo, 10000);

//        showLog("decodeOutIndex:" + outputIndex);
        ByteBuffer outputBuffer;
        byte[] chunkPCM;
        while (outputIndex >= 0) {//每次解码完成的数据不一定能一次吐出 所以用while循环，保证解码器吐出所有数据
            outputBuffer = decodeOutputBuffers[outputIndex];//拿到用于存放PCM数据的Buffer
            chunkPCM = new byte[decodeBufferInfo.size];//BufferInfo内定义了此数据块的大小
            outputBuffer.get(chunkPCM);//将Buffer内的数据取出到字节数组中
            outputBuffer.clear();//数据取出后一定记得清空此Buffer MediaCodec是循环使用这些Buffer的，不清空下次会得到同样的数据
            putPCMData(chunkPCM);//自己定义的方法，供编码器所在的线程获取数据,下面会贴出代码
            mediaDecode.releaseOutputBuffer(outputIndex, false);//此操作一定要做，不然MediaCodec用完所有的Buffer后 将不能向外输出数据
            outputIndex = mediaDecode.dequeueOutputBuffer(decodeBufferInfo, 10000);//再次获取数据，如果没有数据输出则outputIndex=-1 循环结束
        }

    }
```


### 编码

api的使用跟解码的差不多

#### 初始化

``` bash
 public void VideoEncodePrepare() {
        String outputPath = mfileDir + mVideoOutName;
 
        MediaFormat format = MediaFormat.createVideoFormat(MIME_TYPE, mWidth, mHeight);

        // Set some properties.  Failing to specify some of these can cause the MediaCodec
        // configure() call to throw an unhelpful exception.
		format.setInteger(MediaFormat.KEY_COLOR_FORMAT, encoderColorFormat); // 颜色空间的选择是个头疼的机型问题
        format.setInteger(MediaFormat.KEY_BIT_RATE, BIT_RATE);  // 比特率
        format.setInteger(MediaFormat.KEY_FRAME_RATE, FRAME_RATE); // 帧率 （对于h264编码无用?）
        format.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, IFRAME_INTERVAL); // 关键帧

        mediaCodec = null;

        try {
            mediaCodec = MediaCodec.createEncoderByType(MIME_TYPE); // 建议使用 MediaCodec.createByCodecName()
            mediaCodec.configure(format, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);
            mediaCodec.start();

            mMuxer = new MediaMuxer(outputPath, MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4);

        } catch (IOException ioe) {
            throw new RuntimeException("failed init encoder", ioe);
        }

        mTrackIndex = -1;
        mMuxerStarted = false;
    }
```

#### 开始编码

这里省略了喂数据的过程：有个线程解数据放到一个消息队列ArrayBlockingQueue YUVQueue 

```bash
private void doEncodeVideoFromBuffer(MediaCodec encoder, int encoderColorFormat) {

        ByteBuffer[] encoderInputBuffers = encoder.getInputBuffers();
        ByteBuffer[] encoderOutputBuffers = encoder.getOutputBuffers();

        MediaCodec.BufferInfo info = new MediaCodec.BufferInfo(); //  这个对象是用存放编码数据相关信息的（flg ，size）

        int generateIndex = 0;
        int checkIndex = 0;
        int badFrames = 0;


        // The size of a frame of video data, in the formats we handle, is stride*sliceHeight
        // for Y, and (stride/2)*(sliceHeight/2) for each of the Cb and Cr channels.  Application
        // of algebra and assuming that stride==width and sliceHeight==height yields:
        byte[] frameData = new byte[mWidth * mHeight * 3 / 2];
        byte[] frameDataYV12 = new byte[mWidth * mHeight * 3 / 2];
        // Just out of curiosity.
        long rawSize = 0;
        long encodedSize = 0;
        // Save a copy to disk.  Useful for debugging the test.  Note this is a raw elementary
        // stream, not a .mp4 file, so not all players will know what to do with it.


        // Loop until the output side is done.
        boolean inputDone = false;
        boolean encoderDone = false;
        boolean outputDone = false;

        while (!outputDone) {

            if (YUVQueue.size() == 0 && generateIndex < NUMFRAMES) {
                if(isEnd){
                    NUMFRAMES --;
                }
                continue;
            }

            // If we're not done submitting frames, generate a new one and submit it.  By
            // doing this on every loop we're working to ensure that the encoder always has
            // work to do.
            //
            // We don't really want a timeout here, but sometimes there's a delay opening
            // the encoder device, so a short timeout can keep us from spinning hard.
            if (!inputDone) {
                int inputBufIndex = encoder.dequeueInputBuffer(TIMEOUT_USEC);

                if (inputBufIndex >= 0) {
                    long ptsUsec = computePresentationTime(generateIndex);

                    if (generateIndex >= NUMFRAMES) {
                        // Send an empty frame with the end-of-stream flag set.  If we set EOS
                        // on a frame with data, that frame data will be ignored, and the
                        // output will be short one frame.
                        encoder.queueInputBuffer(inputBufIndex, 0, 0, ptsUsec,
                                MediaCodec.BUFFER_FLAG_END_OF_STREAM);
                        inputDone = true;
                        if (VERBOSE) Log.d(TAG, "sent input EOS (with zero-length frame)");

                    } else {
//                        generateFrame(generateIndex, encoderColorFormat, frameData);
//                        ByteBuffer inputBuf = encoderInputBuffers[inputBufIndex];
//                        // the buffer should be sized to hold one full frame
//
//                        inputBuf.clear();
//                        inputBuf.put(frameData);
//                        encoder.queueInputBuffer(inputBufIndex, 0, frameData.length, ptsUsec, 0);
//                        if (VERBOSE) Log.d(TAG, "submitted frame " + generateIndex + " to enc");


                        frameData = YUVQueue.poll();  // 注意格式转换

//                            byte[] yuv420sp = new byte[mWidth * mHeight * 3 / 2];
//                            NV21ToNV12(frameData, yuv420sp, mWidth, mHeight);
//                            frameData = yuv420sp;

                        ByteBuffer inputBuf = encoderInputBuffers[inputBufIndex];
                        // the buffer should be sized to hold one full frame

                        inputBuf.clear();
                        inputBuf.put(frameData);
                        encoder.queueInputBuffer(inputBufIndex, 0, frameData.length, ptsUsec, 0);
                        if (VERBOSE) Log.d(TAG, "submitted frame " + generateIndex + " to enc");

                        mMainHandler.sendEmptyMessage(0);

                    }
                    generateIndex++;
//                    Log.i("bbb", " generateIndex  ==  " + generateIndex);

                } else {
                    // either all in use, or we timed out during initial setup
                    if (VERBOSE) Log.d(TAG, "input buffer not available");
                }
            }
            // Check for output from the encoder.  If there's no output yet, we either need to
            // provide more input, or we need to wait for the encoder to work its magic.  We
            // can't actually tell which is the case, so if we can't get an output buffer right
            // away we loop around and see if it wants more input.
            //
            // Once we get EOS from the encoder, we don't need to do this anymore.
            if (!encoderDone) {
                int encoderStatus = encoder.dequeueOutputBuffer(info, TIMEOUT_USEC);
                if (encoderStatus == MediaCodec.INFO_TRY_AGAIN_LATER) {
                    // no output available yet
                    if (VERBOSE) Log.d(TAG, "no output from encoder available");
                } else if (encoderStatus == MediaCodec.INFO_OUTPUT_BUFFERS_CHANGED) {
                    // not expected for an encoder
                    encoderOutputBuffers = encoder.getOutputBuffers();
                    if (VERBOSE) Log.d(TAG, "encoder output buffers changed");
                } else if (encoderStatus == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
                    // not expected for an encoder
                    MediaFormat newFormat = encoder.getOutputFormat();
                    if (VERBOSE) Log.d(TAG, "encoder output format changed: " + newFormat);

                    if (mMuxerStarted) {
                        throw new RuntimeException("format changed twice");
                    }

                    Log.d(TAG, "encoder output format changed: " + newFormat);

                    // now that we have the Magic Goodies, start the muxer
                    mTrackIndex = mMuxer.addTrack(newFormat);
                    mMuxer.start();
                    mMuxerStarted = true;

                } else if (encoderStatus < 0) {
                    Log.d(TAG, "unexpected result from encoder.dequeueOutputBuffer: " + encoderStatus);
                } else { // encoderStatus >= 0
                    ByteBuffer encodedData = encoderOutputBuffers[encoderStatus];
                    if (encodedData == null) {
                        Log.d(TAG, "encoderOutputBuffer " + encoderStatus + " was null");
                    }

                    if ((info.flags & MediaCodec.BUFFER_FLAG_CODEC_CONFIG) != 0) {
                        // Codec config info.  Only expected on first packet.  One way to
                        // handle this is to manually stuff the data into the MediaFormat
                        // and pass that to configure().  We do that here to exercise the API.

                        MediaFormat format =
                                MediaFormat.createVideoFormat(MIME_TYPE, mWidth, mHeight);
                        format.setByteBuffer("csd-0", encodedData);

                        info.size = 0;

                    } else {
                        // Get a decoder input buffer, blocking until it's available.

                        encoderDone = (info.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0;

                        if ((info.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0)
                            outputDone = true;
                        if (VERBOSE) Log.d(TAG, "passed " + info.size + " bytes to decoder"
                                + (encoderDone ? " (EOS)" : ""));
                    }


                    // It's usually necessary to adjust the ByteBuffer values to match BufferInfo.

                    if (info.size != 0 && ptsQueue.size() > 0) {

                        encodedData.position(info.offset);
                        encodedData.limit(info.offset + info.size);
                        info.presentationTimeUs = ptsQueue.poll();
                        mMuxer.writeSampleData(mTrackIndex, encodedData, info);

                        encodedSize += info.size;

                    }

                    encoder.releaseOutputBuffer(encoderStatus, false);
                }
            }

        }

        if (inputDone) {
            close();
            NativeUtils.endDecodeFrameBySeekTime();
            prevOutputPTSUs = 0;
        }

        if (VERBOSE) Log.d(TAG, "decoded " + checkIndex + " frames at "
                + mWidth + "x" + mHeight + ": raw=" + rawSize + ", enc=" + encodedSize);

        if (checkIndex != NUMFRAMES) {
            Log.d(TAG, "expected " + 120 + " frames, only decoded " + checkIndex);
        }
        if (badFrames != 0) {
            Log.d(TAG, "Found " + badFrames + " bad frames");
        }
    }
```

#### 下一篇则是记录一些开发过程中的坑...