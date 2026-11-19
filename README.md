# CodecScope

**在线使用：https://d24-5e-24dc.github.io/codecscope/**

打开网址即用，把 `.h264` / `.h265` 文件拖进页面即可播放，无需安装任何东西。

基于 [codec2021/codecscope](https://github.com/codec2021/codecscope)（GPL-2.0），
文件取自 https://codec2021.github.io/codecscope/ 的已构建产物（2026-11-19）。

## 修改内容

仅修改 `js/app.js` 中的 `computeFrames()`：帧边界判定改为同时参考 `frame_num`。

原逻辑在 `pic_order_cnt_type == 2` 的码流上失效——解析器把 `slicePoc` 回填为 0，
所有 slice 的 POC 相同，于是整条流被合并成 1 帧，播放第一个 tick 就退出。

现逻辑：同一画面的多个 slice 需 **POC 相同且 frame_num 相同**；
`pic_order_cnt_type = 2` 时 POC 恒为 0，由 `frame_num` 区分画面。

其余文件与上游线上版本逐字节一致。
