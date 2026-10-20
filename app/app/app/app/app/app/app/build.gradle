package com.venice.ffoverlay

object Detector {

    class Hit(val x: Float, val y: Float, val w: Float, val sy: Float)

    fun scan(data: ByteArray, rowStride: Int, w: Int, h: Int,
             thresh: Int, headOffset: Float, max: Int): List<Hit> {

        val x0 = (w * 0.18f).toInt()
        val x1 = (w * 0.82f).toInt()
        val y0 = (h * 0.12f).toInt()
        val y1 = (h * 0.82f).toInt()
        val minRun = (w * 0.028f).toInt().coerceAtLeast(5)
        val maxRun = (w * 0.45f).toInt()
        val off = h * headOffset
        val cx = w / 2f
        val cy = h / 2f

        val cands = ArrayList<Hit>()
        val score = ArrayList<Float>()

        var y = y0
        while (y < y1) {
            var x = x0
            var run = -1
            while (x <= x1) {
                var bright = false
                if (x < x1) {
                    val i = y * rowStride + x * 4
                    bright = (data[i].toInt() and 0xFF) >= thresh &&
                             (data[i + 1].toInt() and 0xFF) >= thresh &&
                             (data[i + 2].toInt() and 0xFF) >= thresh
                }
                if (bright) { if (run < 0) run = x }
                else if (run >= 0) {
                    val len = x - run
                    if (len in minRun..maxRun) {
                        val mx = (run + x) / 2f
                        val hy = y + off
                        cands.add(Hit(mx, hy, len.toFloat(), y.toFloat()))
                        score.add(Math.hypot((mx - cx).toDouble(),
                                             (hy - cy).toDouble()).toFloat())
                    }
                    run = -1
                }
                x++
            }
            y++
        }
        if (cands.isEmpty()) return emptyList()

        val out = ArrayList<Hit>()
        for (i in cands.indices.sortedBy { score[it] }) {
            val c = cands[i]
            var ok = true
            for (k in out) {
                if (Math.abs(k.x - c.x) < w * 0.07f && Math.abs(k.y - c.y) < h * 0.07f) {
                    ok = false; break
                }
            }
            if (ok) { out.add(c); if (out.size >= max) break }
        }
        return out
    }
}
