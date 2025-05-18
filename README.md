# Taiwanese Speech Recognition

## 專案簡介

本專案旨在探索台灣閩南語語音辨識的各種方法。我們利用提供的資料集，嘗試了不同的深度學習模型，包括循環神經網路 (RNN) 和 Transformer 模型 (Whisper)，並對其效能進行評估。

## 資料集說明

### 訓練資料

* **格式**: 3119 個 WAV 音檔，長度不一 (平均約 4 秒)，所有錄音皆為清晰的女性聲音。
* **檔案**: `train-toneless.csv` (提供音檔對應的羅馬字轉寫文本，不包含聲調)。
* **範例 (`train-toneless.csv`)**:


| id | text                                           |
|----|------------------------------------------------|
| 1  | li be e mih kiann lan lan san san long be tsiau tsng |
| 2  | suah ka li tim tioh                           |
| 3  | kiu lang ooh                                  |


### 驗證資料

* 從訓練資料中劃分出 10% 作為驗證集。
* **格式**: 346 個 WAV 音檔，每個約 4 秒長，包含清晰的女性聲音。
* **用途**: 用於模型訓練過程中的效能評估和超參數調整。

### 測試資料

* 格式與 Kaggle 評分上傳格式相似。
* **評估方式**: 詳細說明請見下方的「評分標準」。

### 資料集目錄結構

```
/content/unzipped_files
└── kaldi-taiwanese-asr
    ├── train            # 訓練音檔 (1.wav, 2.wav, ...)
    ├── test             # 測試音檔 (test.wav files)
    ├── train-toneless.csv   # 主要的訓練標註檔 (文本轉寫)
    └── lexicon.txt      # 使用 lexicon.txt 進行音素層級建模的方法可能會用到
```

## 評分標準

1. **評估準則**:
   - 辨識輸出的文本應與提供的**台羅拼音 (Tai-lo)** 文本一致，**不考慮聲調符號**。
   - 輸出範例: `"li be e mih kiann lan lan san san long be tsiau tsng"`

2. **Kaggle 評分指標**:
   - 由於 Kaggle 的限制，評分基於**字元級的平均 Levenshtein 距離**計算。
   - 公式：LevDistance = 插入次數 + 刪除次數 + 替換次數
   - **注意**: 對於語音轉寫任務而言，**詞錯誤率 (WER)** 通常是更合適的評估指標。

## 使用資源

* **執行環境**: **Google Colaboratory (Colab)**
* **硬體**: **A100 GPU**

## 專案任務與結果

### 任務一：循環神經網路 (Recurrent Neural Networks)

* 我們嘗試了使用 RNN 來建立語音辨識模型。
* [查看 Task 1 的 Notebook](https://github.com/ysh1017/Taiwanese_Speech_Recognition/blob/main/(V6)Task_1_Recurrent_Neural_Networks.ipynb)
* **成績**: 不佳

![Task 1 模型結構示意圖](https://github.com/user-attachments/assets/c71b710a-497e-4e88-8209-2622709f340c)

### 任務三：使用 Whisper 模型

本部分探索了基於 Transformer 的預訓練模型 Whisper 在台灣閩南語語音辨識上的應用。近期有許多研究也關注於利用 Whisper 模型進行台語語音辨識的研究。

1. **使用 Whisper large-v3**:
   * 我們直接使用了 OpenAI 提供的 `large-v3` 模型。
   * **資料清理**: 我們對訓練文本進行了初步清理，移除了數字、問號、逗號、聲調符號和連字號。
    ![image](https://github.com/user-attachments/assets/679520ed-134c-4163-b156-0abd3f546211)
   * **Public Score**: 25.27184
    ![Whisper large-v3 Public Score](https://github.com/user-attachments/assets/f25ba924-3665-4b8f-996e-4777968b68c6)

2. **使用 cool-whisper**:
   * `cool-whisper` 是一個基於 Whisper 蒸餾而成的模型，主要針對在國語-英文語碼轉換 ASR進行優化。
   * 該模型使用了 6 萬小時的未標註音訊進行訓練，利用了 `Whisper-large-v2` 和 `Whisper-base` 的知識。
   * [查看 cool-whisper 的 Notebook](https://github.com/user-attachments/assets/3dc7af53-eb47-4772-b94d-bb3c0713c8b5)
   * **資料清理**: 沿用與 `Whisper large-v3` 相同的資料清理步驟 (移除數字、問號、逗號、聲調符號和連字號)。
    ![image](https://github.com/user-attachments/assets/ae0c3fc2-158f-465c-9eaa-9c35fc1fc5b8)
   * **Public Score**: 16.31067
    ![cool-whisper Public Score](https://github.com/user-attachments/assets/ca267e32-d2a5-4280-b96c-685c6a404d3c)

3. **OpenAI Whisper 微調 (Fine-tuning)**:
   * 我們以 `small` 作為基礎模型 (baseline) 進行微調。
   * **驗證集表現**: `Wer` = 8.877381
   
    ![image](https://github.com/user-attachments/assets/a8ade0ba-7bda-4f75-9376-d3f3943a0140)
   * [查看 Fine-tuning 的 Notebook](https://github.com/ysh1017/Taiwanese_Speech_Recognition/blob/main/V2_fine_tune_whisper.ipynb)
   * **Private Score**: 2.21399
   ![image](https://github.com/user-attachments/assets/e6f5fd85-c402-414f-9768-eec4b29f4494)


   * **與學術研究的比較**: 我們的微調實驗與近期一些學術研究方向相似，皆著重於客製化 Whisper 模型以提升台語辨識效能。例如，謝岳che、呂克明和呂仁園在 **ROCLING 2023** 會議上發表的論文[〈**運用基於生成預訓練轉換器架構的 OpenAI Whisper 多語言語音辨識引擎之台語及華語語音辨識之實作**〉](https://ndltd.ncl.edu.tw/cgi-bin/gs32/gsweb.cgi/ccd=2XiMMk/search?s=id=%22111CGU05392012%22.&openfull=1&setcurrent=0)中，也探討了對 Whisper 模型進行微調以辨識台語和華語的可能性。

   * **論文觀點融入**: 該論文使用了 Hugging Face 提供的 Whisper Medium 和 Large-v2 模型，並利用 CommonVoice 的台語資料集以及網路蒐集的台語連續劇影片和字幕檔 (約 800 小時) 進行微調，最佳的字元錯誤率 (CER) 達到 50.7%。他們的研究也指出，儘管微調能有效提升 Whisper 的台語辨識能力，但目前的 CER 指標可能無法完全反映辨識效果，部分原因是台語口語與華語書寫的轉換存在多對應關係，且現有的台語語料庫規模仍然有限。

   * **與本研究的異同**: 值得注意的是，上述論文目標是使 Whisper 輸出繁體漢字，而我們的目標是基於**台羅拼音**的語音辨識，這代表著標註和評估方式上的差異。他們使用 CER 作為評估指標，而我們則採用 **Levenshtein 距離**。儘管如此，他們在資料收集和模型微調上的經驗，以及對於台語語音辨識挑戰的觀察 (例如資料稀疏性、評估指標的局限性)，都為我們的研究提供了寶貴的參考。

   * **本研究的發現**: 我們的實驗也觀察到，即使是以 `cool-whisper` 作為基線進行微調，仍然面臨一些挑戰，這與上述論文中提到的困難相呼應。儘管我們的評估指標和目標輸出與他們的有所不同，但提升台語語音辨識的難度仍然存在。

## 結論與未來方向

* 最後成功取得 **Private Score**: 2.21399
    ![image](https://github.com/user-attachments/assets/f07e90ec-0d91-4aa2-8675-f2e7d811878a)

* 透過實驗，我們發現基於 Transformer 的 Whisper 模型在台灣閩南語語音辨識任務上展現了較好的效能，特別是 `cool-whisper` 作為針對台灣口音優化的模型，取得了明顯的進展。
* 我們也借鑒了學術研究的經驗，理解到台語語音辨識仍面臨資料量、評估指標等多方面的挑戰。如同謝岳che 等人的研究指出，台語語音與華語文字的轉換具有複雜性，且現有資源的限制也影響了模型的效能。
* 後續可以嘗試以下方向：
    * 更精細的資料清理和預處理。
    * 探索不同的 Whisper 微調策略和參數設定。
    * 嘗試使用更大的預訓練模型或進行模型融合。
    * 考慮加入聲調資訊的建模。
    * 評估在真實應用場景中的效能。
    * 參考相關學術研究，例如擴展訓練資料集、設計更有效的評估指標等。
