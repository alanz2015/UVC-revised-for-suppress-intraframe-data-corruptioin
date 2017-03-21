# UVC-revised-for-suppress-intraframe-data-corruptioin
Due to non-guarantee bandwidth for BULK mode transmitted over USB 3.0 port, so when transmit high definition raw stream (YUV) over the link may have intra-frame data corruption, then display w/ micro-blocks, so add some work around to suppress this micro-block issues on screen.

Apply the following patch from Internet, 
1.	diff --git a/drivers/usb/host/xhci.h b/drivers/usb/host/xhci.h
2.	index 8820871b..0322430 100644
3.	--- a/drivers/usb/host/xhci.h
4.	+++ b/drivers/usb/host/xhci.h
5.	@@ -1238,7 +1238,7 @@ union xhci_trb {
6.	  * since the command ring is 64-byte aligned.
7.	  * It must also be greater than 16.
8.	  */
9.	-#define TRBS_PER_SEGMENT  64
10.	+#define TRBS_PER_SEGMENT  256
11.	/* Allow two commands + a link TRB, along with any reserved command TRBs */
12.	#define MAX_RSVD_CMD_TRBS (TRBS_PER_SEGMENT - 3)
13.	#define TRB_SEGMENT_SIZE  (TRBS_PER_SEGMENT*16)


1.	diff --git a/drivers/media/usb/uvc/uvcvideo.h b/drivers/media/usb/uvc/uvcvideo.h
2.	index af505fd..e5f36ef 100644
3.	--- a/drivers/media/usb/uvc/uvcvideo.h
4.	+++ b/drivers/media/usb/uvc/uvcvideo.h
5.	@@ -112,9 +112,9 @@
6.	 #define DRIVER_VERSION           "1.1.1"
7.	 
8.	 /* Number of isochronous URBs. */
9.	-#define UVC_URBS          5
10.	+#define UVC_URBS          76
11.	 /* Maximum number of packets per URB. */
12.	-#define UVC_MAX_PACKETS          32
13.	+#define UVC_MAX_PACKETS          64
14.	 /* Maximum number of video buffers. */
15.	 #define UVC_MAX_VIDEO_BUFFERS    32
16.	 /* Maximum status buffer size in bytes of interrupt URB. */

================================================================
Then, modify static int uvc_video_decode_start(struct uvc_streaming *stream,
		struct uvc_buffer *buf, const __u8 *data, int len) / uvc_video.c :
    
	/* Mark the buffer as bad if the error bit is set. */
	if (data[1] & UVC_STREAM_ERR) {
		uvc_trace(UVC_TRACE_FRAME, "Marking buffer as bad (error bit "
			  "set).\n");
		buf->error = 1;
		// Add work around to suppress corrupted frame data
		buf->error = 0;
		buf->state = UVC_BUF_STATE_QUEUED;
		buf->bytesused = 0;
		return -ENODATA;
		
	}
