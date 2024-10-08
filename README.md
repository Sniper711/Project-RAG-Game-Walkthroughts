# 遊戲大神們需要的RAG對話機器人 - 最新的遊戲通關秘笈與彩蛋都在這   
NVIDIA AI-AGENT 夏季訓練營
項目名稱：AI-AGENT 夏季訓練營 — RAG 智能對話機器人  
報告日期：2024年8月18日  
項目負責人：[林耀南, SNIPER711 LIN, Davis Lin]  
<img width="1231" alt="7" src="https://github.com/user-attachments/assets/8b97a2c1-ff1e-467d-ac9c-482ce0cfe526">


# 專案概述 - 遊戲大神們需要的RAG對話機器人 - 最新的遊戲通關秘笈與彩蛋都在這  
硬核遊戲玩家們, 不是在打遊戲, 就是在搜尋遊戲五花八門的通關秘笈與攻略的路上.  
常常遊戲已經破台了, 還因為剛找到完美通關, 或者剛發現遊戲彩蛋的方法, 而重新玩一遍特定的章節任務, 即時在 social media 分享成果.  
網路上琳瑯滿目的遊戲攻略網站, 找起來真的費力, 得要猜中搜尋關鍵字, 要搜到即時更新的網站, 還要花時間閱讀比對.  
若能創造一個自動爬文, 把網上最新的遊戲 walkthroughs 時時收集抓來做成 RAG, 再以大型語言模型 LLM 做成問答機器人, 對遊戲玩家會很方便, 對遊戲公司想推廣自己的遊戲也很有利.  
![Game Guide Books](https://github.com/user-attachments/assets/d323e0c5-ed93-4346-87ae-f4b71b8a7641)




# 技術方案與實施步驟  
* 服務選擇：首推 NVIDIA NIM 服務. 有玩過 LLM 模型的就知道, 到處申請 Keys 有多不方便. 而在 NVIDIA NIM 裡面, 只要一隻 NVIDIA API KEY 就能選用大部分的推理模型, 例如：  
  * 微軟 phi-3-medium-128k-Instruct  
  * Meta llama-3-405b-Instruct, llama-3.1-70b-Instruct, llama-3.1-8b-Instruct  
  * NVIDIA nemotron-4-340b-reward, nemotron-codstral-7b-v0.1  
  * NV-Mistralai mistral-nemo-12b-Instruct  
  * Mistralai mathstral-7b-v0.1, mamba-codstral-7b-v0.1, codestral-22b-Instruct-v0.1  
  * Google gemma-2-2b-it, shieldgemma-9b  
  * Writer palmyra-fin-70b-32k, palmyra-med-70b, palmyra-med-70b-32k  
  * Rakuten rakutenai-7b-Instruct, rakutenai-7b-chat  
  * Thudm chatglm3-6b  
  * Baichuan-int baichuan2-13b-chat  
除了以上的推理模型 Reasoning, 在下圖左邊分類上, 還有視覺 Vision、視覺設計 Vision Design、檢索 Retrival、語音 Speech、生物學 Biology、模擬 Simulation、遊戲 Gaming、醫療 Healthcare、工業 Industral 的模型與SDK能任選吃到飽. 其中還有不少是提供 docker images, 如入寶山一樣.  
[NVIDIA NIM 服務網址, 這次才知道對所有人開放, 用 email 註冊即刻能用](https://build.nvidia.com/explore/reasoning)  
  <img width="1000" alt="截圖 2024-08-17 21 19 23" src="https://github.com/user-attachments/assets/374ebab8-290e-4252-abce-6b2a7875bdb6">

* 模型選擇：  
  * 我這次 LLM 模型用 ai-mixtral-8x7b-instruct, 這是一個預訓練的混合專家 (Mixture of Experts, MoE) 模型, 在一開始還沒有想好題目要做自然語言處理、圖像辨識、還是邏輯推理的時候, 模型選 ai-mixtral-8x7b-instruct 是個安全的選擇.  
  * RAG 自然是要用 Langchain 了, langchain 的線上教學資源與其 API 都很完整, 還有 langsmith 能線上 dashboard 分析.  

* 數據的構建：  
  * 我先選我花過很多時間玩的GTAV遊戲, 拿 [GTABase](https://www.gtabase.com/grand-theft-auto-v/missions/) 網站當第一個嘗試對象, 建立我的即時檢索數據庫.
  <img width="771" alt="截圖 2024-08-18 07 53 46" src="https://github.com/user-attachments/assets/68b4c0f0-1620-4c95-849d-87a95d65e109">
  <div align=center> <img width="478" alt="截圖 2024-08-18 08 01 27" src="https://github.com/user-attachments/assets/d487e964-10c3-4412-98a4-a3a11e9c37e4">

  * 大部分有用的 walkthroughs 資料在網頁的下一層, 所以要改寫一個能在網頁下探更多層超連結的爬蟲, 也要改寫不被網站發現而被屏蔽的爬蟲.  
  * 抓到的文字, Chunk 分段用簡單的1000個字母數切斷一次, 無重疊 overlap 字母數. 若效果不好, 才考慮用單詞數切斷, 加重疊 overlap 單詞數.  
  * 為避免出錯, 增加一段 check_texts_format(texts): 確保收到的都是文字, 才進行下一步 Embedding 處理.
  * Embedding 的模型, 我選擇 NVIDIAEmbeddings 工具類, 調用 NVIDIA NIM 中的 ai-embed-qa-4 向量化模型. 這是一個能 GPU 加速的 embedding 模型, 而要處理的資料多的時候, embedding 的速度就很重要, 使用 NVIDIA NIM 中的 ai-embed-qa-4 向量化模型, 之後我容易能繼續改成 GPU 加速.

***	功能整合（进阶版RAG必填）：  介绍进阶的语音功能、Agent功能、多模态等功能的整合策略与实现方法。


# 實施步驟：
* 環境搭建：
  * 用 Windows 電腦, 安裝 Miniconda [Miniconda官網下載點](https://docs.conda.io/en/latest/miniconda.html)
  * 安裝完之後, 在 Windows 打開 Anaconda Powershell Prompt 終端機.
    <img width="425" alt="截圖 2024-08-18 08 06 53" src="https://github.com/user-attachments/assets/cb007005-c33a-405a-bc17-036d0b07c7ea">
  * 在打開的終端機視窗裡：
    * 建立虛擬環境, 指令 conda create --name ai_endpoint python=3.8
    * 進入虛擬環境, 指令 conda activate ai_endpoint
    * 安裝 nvidia_ai_endpoint 工具, 指令 pip install langchain-nvidia-ai-endpoints
    * 安裝 Jupyter Lab, 指令 pip install jupyterlab
    * 安裝 langchain_core, 指令 pip install langchain_core
    * 安裝 langchain, 指令 pip install langchain  
    * 安裝 matplotlib, 指令 pip install matplotlib  
    * 安裝 Numpy, 指令 pip install numpy  
    * 安裝 faiss (這裡沒有GPU可先安裝CPU版本, 之後再換上GPU來高速運算), 指令 pip install faiss-cpu==1.7.2  
    * 安裝 OPENAI 庫, 指令 pip install openai  
    * 打開 Jupyter Lab 開始寫code, 指令 jupyter-lab  
  * 申請 NVIDIA NIM 帳號, 拿 NVIDIA_API_KEY：  
    * 申請 NVIDIA NIM 帳號：  
      打開 [NVIDIA NIM](https://build.nvidia.com/explore/discover) 在右上角 login 點一下, 在跳出的視窗輸入自己的 email 申請帳號  
      <img width="1025" alt="截圖 2024-08-18 08 21 03" src="https://github.com/user-attachments/assets/4613cacf-a789-4b74-9b1f-b6912a8ca217">  
    * 拿 NVIDIA_API_KEY：  
      畫面中任選一個模型按下去, 跳轉模型介紹頁面的右面有個 Get API Key 按下去, 會跳出 Generate Key 視窗.  
      <img width="1001" alt="截圖 2024-08-18 08 27 38" src="https://github.com/user-attachments/assets/c0d7e3b6-66a6-4ec6-aca8-315d7d3eb405">
      按一下 Generate Key, 把產生的 NVIDIA_API_KEY 複製起來存好 (有效期限一年), 這樣就能開始用 NVIDIA NIM 裡面的許多服務了.
      <img width="993" alt="截圖 2024-08-18 08 29 31" src="https://github.com/user-attachments/assets/fd57836a-c0ee-4fd8-9733-0ed42f2d80ee">
      NVIDIA_API_KEY 只需要申請一支, 當你再次申請時, 前一支 Key 自動失效.  
      NVIDIA NIM 服務裡面所有的 models 都共用同一支 Key, 方便管理.  

*	代碼實現(只列出關鍵代碼)、測試與調校優化、集成與部署、項目成果展示：
  *	匯入需要的函式庫, 輸入 NVIDIA_API_KEY 以使用 NVIDIA NIM 的模型.
    <img width="671" alt="nvidia api key" src="https://github.com/user-attachments/assets/6622fd56-6058-424b-9f7c-89391f013b50">
  *	做一個多層網路的爬蟲程式, 見圖中的 def html_document_loader, def get_all_links (能進網頁多層連結的深度爬蟲), def load_all_linked_documents.  
    過程中, 爬蟲程式被 GTAVBase 網站擋住, 所以加做一個 headers 偽裝成真人上網瀏覽, 見圖中的 headers.  
    <img width="733" alt="2" src="https://github.com/user-attachments/assets/e945806c-4bc6-4164-9721-4dd6a23978f4">
  * 定義 embeddings 數據向量化的函式, 能 depth 設定要抓到網站的第幾層深度
    <img width="472" alt="3" src="https://github.com/user-attachments/assets/49da7127-b24c-4100-b105-6deeb5efd2c2">
  * 先做一個能刪除先前 embedding 資料的程式 (視需要選用), 這是因為寫程式過程中嘗試 embeddings, 累積了一些其他網站的資料需要被清除.
    然後就是漫長等待 embedding 的時間. 這段 code 我之後必須改用 GPU 加速.  
    <img width="356" alt="4" src="https://github.com/user-attachments/assets/dde5a7e5-8477-4c0a-9221-200e9717a27a">
  * 先測一下 LLM 抓 RAG 資料回答遊戲某個任務的秘笈, 看來是成功了  
    <img width="623" alt="5" src="https://github.com/user-attachments/assets/9154f590-8618-4a0f-8cba-eb286a31e409">
  * 那麼就開始做問答機器人了, 用 Gradio, 我想讓他開在一個新的瀏覽器視窗, 左邊是遊戲縮圖, 右邊做遊戲秘笈的文字問答視窗  
    <img width="625" alt="6" src="https://github.com/user-attachments/assets/d5ece68b-311d-499e-a6f5-9c580c839fe6">
  * 將醬～ 這就是我的專案成果：建立RAG對話機器人 - 查最新的遊戲通關秘笈與彩蛋  
    <img width="1231" alt="7" src="https://github.com/user-attachments/assets/c28a4084-cd50-461e-9b05-2905cf720213">

# 問題與解決方案：  
*	遇過的主要問題：  
  * 樹狀分支的網頁, 需要多層次的網路爬蟲.  
  * 網站屏蔽我的爬蟲, 無法挖掘資料.    
  * 需要清除舊的 embedding 資料. 在初期多次試錯之後, 程式已經能正確跑之後, 需要乾淨的重建 embedding 資料庫.  
  * CPU 跑 Embedding 太慢太久.  
  * RAG 對話機器人的 問答方式, 對話機器人的 UI 設定.  
  * 初期多次試錯時, Gradio port 5000 當掉了.  
* 解決措施： 
  * 前面幾個問題, 請參考以上我的代碼截圖與說明.  
  * 最後一個問題 port 換到 7860 可解.  
    
# 項目總結與展望：  
  * 這個自創的專案是個開始, 這是我之後要繼續開發客服相關的 RAG 對話機器人的前期準備.   
  * 客服市場的潛力很大, 因為隱私考量所以適合在本地自己的設備上跑, 這個市場AI從雲端真的落地開花的關鍵.  
  * 我喜歡AI客服這種能 scalable 的市場, 投入最開始的一份努力, 微調一下就能 scalable 給許多不同的客戶使用.   
  * 而不喜歡來100個客人就要投入100份成本的市場.  

# 其他：
  這次 NVIDIA AI-AGENT 夏季訓練營讓我獲益良多, 上課教材給的是基礎的程式碼, 環境搭建說明與基本 code 的寫法.  
  而最有收穫的是課後要自己成立一個專案, 思考應用場景, 程式碼缺少的許多拼圖要怎麼做, 遇到問題要迎頭解決還是技巧繞開.  
  雖然每天上課只有兩小時, 但每天課前課後幾乎整天都在自習.  
  要稱讚一下 NVIDIA 不休息的助教群們, 雖然每天上課也是兩小時, 但是助教們在群裡幾乎是從早起床到晚上12點都在即時回覆, 還有鼓勵大家完成課後專案. 給拍拍.    
