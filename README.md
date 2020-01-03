# IOT_Project: 車牌辨識行車紀錄器
設計一套行車專用配件，利用鏡頭記錄行車過程，在車禍時，利用震動感測器模組，可判斷出車禍發生的時機，利用這個時間資訊，讓樹苺派知道此段時間點的影片檔是不能刪除的，再利用Intel的Pretrained Models辨識車輛車牌，透過紀錄車牌號碼，對於遇到肇事逃逸的車禍來說，得以使車主快速獲取車牌資訊，不用慢慢看影片去找車牌，並利用手機熱點在連網時將車牌資訊上傳到雲端，方便車主檢舉或給予警方相關資料。<br /><br />
Input：鏡頭 震動感測器模組。<br />
Output：行車紀錄的影片記錄於樹苺派中，辨識完車牌的圖片及結果則存於樹苺派及Google雲端的文件中。<br />
Video Demo: https://drive.google.com/open?id=1eMStMv6OOEz_Fmdua6y77xWdLzWBTzqE <br /><br />
所需材料：<br />
樹莓派及電源 * 1 <br />
麵包板 * 1 <br />
鏡頭 * 1 <br />
SW-420 常閉型震動振動感測器模組 * 1 <br />
杜邦線 <br />
# 步驟一：啟用 Google 雲端硬碟 API
參考教學：https://blog.gtwang.org/programming/automation-of-google-ocr-using-python-tutorial/ <br />
進入 https://console.developers.google.com/flows/enableapi?apiid=drive <br />


xxxx
