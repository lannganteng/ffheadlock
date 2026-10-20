package com.venice.ffoverlay

import android.app.*
import android.content.Context
import android.content.Intent
import android.content.pm.ServiceInfo
import android.graphics.Color
import android.graphics.PixelFormat
import android.hardware.display.DisplayManager
import android.hardware.display.VirtualDisplay
import android.media.Image
import android.media.ImageReader
import android.media.projection.MediaProjection
import android.media.projection.MediaProjectionManager
import android.os.*
import android.view.Gravity
import android.view.WindowManager
import android.webkit.JavascriptInterface
import android.webkit.WebView
import android.webkit.WebViewClient

class OverlayService : Service() {

    private lateinit var wm: WindowManager
    private lateinit var overlay: WebView
    private lateinit var panel: WebView
    private var panelLp: WindowManager.LayoutParams? = null

    private val ui = Handler(Looper.getMainLooper())
    private val CH = "ffhl"

    private var proj: MediaProjection? = null
    private var vd: VirtualDisplay? = null
    private var reader: ImageReader? = null
    private var capW = 0
    private var capH = 0
    private var scaleX = 1f
    private var scaleY = 1f
    private var pix: ByteArray? = null
    private var lastScan = 0L

    private var on = true
    private var guide = true
    private var debug = true
    private var lead = 0f
    private var radius = 200f
    private var thresh = 190
    private var headOffset = 0.035f
    private var offX = 0f
    private var offY = 0f
    private var found = false

    override fun onCreate() {
        super.onCreate()

        if (Build.VERSION.SDK_INT >= 26) {
            getSystemService(NotificationManager::class.java).createNotificationChannel(
                NotificationChannel(CH, "FF Headlock", NotificationManager.IMPORTANCE_LOW))
        }
        val n = Notification.Builder(this, CH)
            .setContentTitle("FF Headlock AKTIF")
            .setContentText("kunci kepala musuh - kamu yang nembak")
            .setSmallIcon(android.R.drawable.ic_menu_view)
            .setOngoing(true).build()
        if (Build.VERSION.SDK_INT >= 29)
            startForeground(1, n, ServiceInfo.FOREGROUND_SERVICE_TYPE_MEDIA_PROJECTION)
        else startForeground(1, n)

        wm = getSystemService(Context.WINDOW_SERVICE) as WindowManager

        overlay = WebView(this).apply {
            setBackgroundColor(Color.TRANSPARENT)
            settings.javaScriptEnabled = true
            settings.allowFileAccess = true
            webViewClient = WebViewClient()
            loadUrl("file:///android_asset/overlay.html")
        }
        val of = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or
                 WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE or
                 WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS or
                 WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN
        val olp = WindowManager.LayoutParams(
            WindowManager.LayoutParams.MATCH_PARENT,
            WindowManager.LayoutParams.MATCH_PARENT,
            WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,
            of, PixelFormat.TRANSLUCENT
        ).apply { gravity = Gravity.TOP or Gravity.START }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P)
            olp.layoutInDisplayCutoutMode =
                WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES
        wm.addView(overlay, olp)

        panel = WebView(this).apply {
            setBackgroundColor(Color.TRANSPARENT)
            settings.javaScriptEnabled = true
            addJavascriptInterface(Bridge(), "Native")
            loadUrl("file:///android_asset/panel.html")
        }
        val d = resources.displayMetrics.density
        val plp = WindowManager.LayoutParams(
            (272 * d).toInt(), (232 * d).toInt(),
            WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,
            PixelFormat.TRANSLUCENT
        ).apply { gravity = Gravity.TOP or Gravity.START; x = 24; y = 70 }
        panelLp = plp
        wm.addView(panel, plp)

        pushCfg()
    }

    private fun startCapture(code: Int, data: Intent) {
        val mgr = getSystemService(MediaProjectionManager::class.java)
        proj = mgr.getMediaProjection(code, data) ?: return
        val dm = resources.displayMetrics
        capW = (dm.widthPixels / 3).coerceAtLeast(240)
        capH = (dm.heightPixels / 3).coerceAtLeast(240)
        scaleX = dm.widthPixels.toFloat() / capW
        scaleY = dm.heightPixels.toFloat() / capH

        reader = ImageReader.newInstance(capW, capH, PixelFormat.RGBA_8888, 2)
        vd = proj!!.createVirtualDisplay("cap", capW, capH,
            (dm.densityDpi / 3).coerceAtLeast(80),
            DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
            reader!!.surface, null, ui)

        reader!!.setOnImageAvailableListener({ r ->
            val img = r.acquireLatestImage() ?: return@setOnImageAvailableListener
            try { onFrame(img) } catch (t: Throwable) {} finally { img.close() }
        }, ui)
    }

    private fun onFrame(img: Image) {
        val now = SystemClock.uptimeMillis()
        if (now - lastScan < 26) return
        lastScan = now

        val p = img.planes[0]
        val buf = p.buffer
        val need = p.rowStride * capH
        if (pix == null || pix!!.size < need) pix = ByteArray(need)
        val arr = pix!!
        buf.rewind()
        buf.get(arr, 0, minOf(need, buf.remaining()))

        val hits = Detector.scan(arr, p.rowStride, capW, capH, thresh, headOffset, 3)
        found = hits.isNotEmpty()
        if (!on) { push("FFOverlay.targets([])"); return }

        val d = resources.displayMetrics.density
        val json = hits.joinToString(",") {
            "[${it.x * scaleX / d},${it.y * scaleY / d},${it.w * scaleX / d}]"
        }
        push("FFOverlay.targets([$json])")
    }

    private fun push(js: String) = overlay.post { overlay.evaluateJavascript(js, null) }

    private fun pushCfg() {
        val d = resources.displayMetrics.density
        push("FFOverlay.cfg({radius:${radius / d},offX:${offX / d},offY:${offY / d}," +
             "guide:$guide,debug:$debug,lead:$lead,on:$on})")
    }

    inner class Bridge {
        @JavascriptInterface fun set(k: String, v: Float) {
            when (k) {
                "radius" -> radius = v
                "thresh" -> thresh = v.toInt()
                "hoff"   -> headOffset = v
                "lead"   -> lead = v
                "offX"   -> offX = v
                "offY"   -> offY = v
            }
            pushCfg()
        }
        @JavascriptInterface fun toggle(k: String, v: Boolean) {
            when (k) {
                "on"    -> on = v
                "guide" -> guide = v
                "dbg"   -> debug = v
            }
            pushCfg()
        }
        @JavascriptInterface fun drag(dx: Float, dy: Float) {
            val lp = panelLp ?: return
            lp.x += dx.toInt(); lp.y += dy.toInt()
            ui.post { wm.updateViewLayout(panel, lp) }
        }
        @JavascriptInterface fun info(): String = """{"found":$found,"on":$on}"""
    }

    override fun onStartCommand(i: Intent?, f: Int, s: Int): Int {
        if (i?.hasExtra("data") == true) {
            val code = i.getIntExtra("code", 0)
            @Suppress("DEPRECATION")
            val data = i.getParcelableExtra<Intent>("data")
            if (data != null) startCapture(code, data)
        }
        return START_STICKY
    }

    override fun onDestroy() {
        try { vd?.release() } catch (t: Throwable) {}
        try { proj?.stop() } catch (t: Throwable) {}
        try { wm.removeView(overlay) } catch (t: Throwable) {}
        try { wm.removeView(panel) } catch (t: Throwable) {}
        super.onDestroy()
    }

    override fun onBind(i: Intent?) = null
}
