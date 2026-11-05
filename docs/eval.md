# 评测集使用说明

## 为什么要有它

换模型之前先有分数。没有分数的"感觉更准"不算数。

## 目录结构

```
eval/
  score.py              打分（CER / WER / 关键词命中）
  run.py                跑一个模型，输出对比表
  compare.py            对比多次结果
  convert_ct2.py        HF 模型 → CTranslate2（产品用这个格式）
  fetch_candidates.py   下载候选模型（走 hf-mirror，分块并发）
  cleanup.py            清理测试产物，只留选中的包
  cases/
    th/0001_rimo_mv/
      audio.mp4
      reference.txt     人工转写真值
      meta.json         可选：keywords
  results/              每次评测的分数
```

## 用法

```powershell
# 基线（现网 large-v3）
python eval/run.py --engine whisper --model large-v3 --lang th --tag baseline

# 候选模型（HF 格式，需 transformers）
python eval/run.py --engine hf --model E:\path\to\hf-model --lang th --tag cand-hf

# 候选模型（已转 CT2，和产品一致）
python eval/run.py --engine ct2 --model E:\path\to\ct2 --lang th --tag cand-ct2

# 对比
python eval/compare.py
```

## 踩过的坑（务必看）

**1. 真值必须对着音频核对**
最早一版泰语真值用的是"应该说的稿子"，结果三个完全不同的模型（通用 large-v3、
biodatlab 微调、Pathumma 微调）**错在完全相同的位置**（สร้าง→สั่ง、กำหนด→กำหนัด）。
这种一致性说明不是模型的问题，而是真值/音频对不上。真值要边听边敲。

**2. 拉丁借词会污染 CER**
真值写 `Rimo`、`MV`，而泰语模型倾向输出 `รีโม่`、`เอ็มวี`。字符级比较会把这种
**合理差异**算成错误，导致泰语专用模型被低估。打分前应把借词统一，或单独统计。

**3. 一个 8 秒样本不足以定论**
每语种至少 5 段、不同说话人/语速/背景，否则换个片段结论就翻转。

## 已知结论（2026-11）

| 语种 | 基线 CER | 候选 | 结论 |
|---|---|---|---|
| th | 17.4%（单片段，含借词干扰） | biodatlab 22.6%、Pathumma 23.5% | **不换**，先修真值再评 |

模型候选、体积、许可都在 `probe_models.py` 里可查。
