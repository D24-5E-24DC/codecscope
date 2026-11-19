# CodecScope（修复版静态副本）

基于 [codec2021/codecscope](https://github.com/codec2021/codecscope)（GPL-2.0），
文件取自 https://codec2021.github.io/codecscope/ 的已构建产物（2026-11-19）。

## 修改内容

仅修改 `js/app.js` 中的 `computeFrames()`：帧边界判定改为同时参考 `frame_num`。

原逻辑在 `pic_order_cnt_type == 2` 的码流上失效——解析器把 `slicePoc` 回填为 0，
所有 slice 的 POC 相同，于是整条流被合并成 1 帧，播放第一个 tick 就退出。

现逻辑：同一画面的多个 slice 需 **POC 相同且 frame_num 相同**；
`pic_order_cnt_type = 2` 时 POC 恒为 0，由 `frame_num` 区分画面。

其余文件与上游线上版本逐字节一致。

## 验证

- hicar-decoder_befor.h264（poc_type=2，503 slice）：可正常播放至末帧（501/502）。
- AA-decoder_befor.h264（poc_type=2，948 slice）：同上。
- carplay_800x480.H264（每帧 2 slice，850 slice）：仍精确切为 425 帧，未拆帧。
- hicar-encoder_after.h264（poc_type=0，139 slice）：行为不变，正常播放。

## 部署

把本目录全部内容上传到 GitHub 仓库根目录，Settings → Pages →
Source 选 "Deploy from a branch"，分支 main，目录 / (root)。
