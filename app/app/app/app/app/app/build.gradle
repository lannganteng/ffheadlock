package com.venice.ffoverlay

import android.Manifest
import android.app.Activity
import android.content.Intent
import android.content.pm.PackageManager
import android.media.projection.MediaProjectionManager
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.provider.Settings
import android.widget.Button
import android.widget.LinearLayout
import android.widget.TextView
import android.widget.Toast

class MainActivity : Activity() {

    private val RC_OVL = 11
    private val RC_PROJ = 12
    private lateinit var status: TextView

    override fun onCreate(s: Bundle?) {
        super.onCreate(s)

        if (Build.VERSION.SDK_INT >= 33 &&
            checkSelfPermission(Manifest.permission.POST_NOTIFICATIONS)
            != PackageManager.PERMISSION_GRANTED) {
            requestPermissions(arrayOf(Manifest.permission.POST_NOTIFICATIONS), 9)
        }

        val ll = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setPadding(60, 140, 60, 60)
        }
        ll.addView(TextView(this).apply {
            text = "FF HEADLOCK OVERLAY"
            textSize = 20f
        })
        ll.addView(TextView(this).apply {
            text = "Tekan tombol 1 lalu tombol 2. Kamu yang kontrol aim dan tembak."
            textSize = 12f
            setPadding(0, 12, 0, 34)
        })

        ll.addView(btn("1.  Izin tampil di atas aplikasi") {
            startActivityForResult(Intent(
                Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                Uri.parse("package:$packageName")), RC_OVL)
        })
        ll.addView(btn("2.  MULAI") {
            val mgr = getSystemService(MediaProjectionManager::class.java)
            startActivityForResult(mgr.createScreenCaptureIntent(), RC_PROJ)
        })

        status = TextView(this).apply { textSize = 13f; setPadding(0, 44, 0, 0) }
        ll.addView(status)
        setContentView(ll)
        refresh()
    }

    private fun btn(t: String, f: () -> Unit) = Button(this).apply {
        text = t
        textSize = 15f
        setPadding(0, 26, 0, 26)
        setOnClickListener { f() }
    }

    override fun onActivityResult(rc: Int, res: Int, data: Intent?) {
        super.onActivityResult(rc, res, data)
        if (rc == RC_PROJ && res == RESULT_OK && data != null) {
            val i = Intent(this, OverlayService::class.java)
            i.putExtra("code", res)
            i.putExtra("data", data)
            if (Build.VERSION.SDK_INT >= 26) startForegroundService(i) else startService(i)
            Toast.makeText(this, "Headlock AKTIF - buka Free Fire MAX", Toast.LENGTH_LONG).show()
        }
        refresh()
    }

    private fun refresh() {
        status.text = "izin overlay : " +
            (if (Settings.canDrawOverlays(this)) "OK" else "BELUM - tekan tombol 1")
    }
}
