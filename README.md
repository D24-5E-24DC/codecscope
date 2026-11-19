# CodecScope

**在线使用：https://d24-5e-24dc.github.io/codecscope/**

打开网址即用，把 `.h264` / `.h265` 文件拖进页面即可播放，无需安装任何东西。

基于 [codec2021/codecscope](https://github.com/codec2021/codecscope)（GPL-2.0），
文件取自 https://codec2021.github.io/codecscope/ 的已构建产物（2026-11-19）。

## 修改内容

仅修改 `js/app.js`：新增 `readFirstMb()`，并改动 `computeFrames()` 的帧边界判定。

**其一，优先使用 spec 级帧边界。** AVC slice header 的第一个语法元素
`first_mb_in_slice` 为 0，当且仅当这是新画面的第一个 slice（宏块地址在画面内唯一）。
它不依赖 POC 与 frame_num，所以每帧都是 IDR（POC 和 frame_num 同时归零）的码流也能正确切帧。
读取失败时整条流回退到下面的 POC + frame_num 逻辑，不做逐 slice 混用。

**其二，POC + frame_num 兜底。** 原逻辑在 `pic_order_cnt_type == 2` 的码流上失效——
解析器把 `slicePoc` 回填为 0，所有 slice 的 POC 相同，于是整条流被合并成 1 帧，
播放第一个 tick 就退出。兜底逻辑要求同一画面的多个 slice **POC 相同且 frame_num 相同**。

其余文件与上游线上版本逐字节一致。
