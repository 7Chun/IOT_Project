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
參考教學：[Python 使用 Google 雲端硬碟 API 自動進行文字辨識教學](https://blog.gtwang.org/programming/automation-of-google-ocr-using-python-tutorial/) <br />
1.進入[Google Drive API 註冊網頁](https://console.developers.google.com/flows/enableapi?apiid=drive )<br />
2.選擇建立專案。<br />
3.前往將憑證新增至你的專案，並使用Google Drive API，呼叫來源選擇其他使用者介面，存取資料勾選使用者資料。<br />
4.設定OAuth2.0同意畫面，並選擇外部，再設定應用程式名稱。<br />
5.建立OAuth2.0用戶端ID，並輸入剛才的應用程式名稱。<br />
6.下載憑證，並儲存為`client_id.json`。<br />
# 步驟二：安裝 Google Client Library
1.進入console輸入指令<br />
`#pyhton2`<br />

    $pip install --upgrade google-api-python-client

`#pyhton3`<br />

    $pip3 install --upgrade google-api-python-client

# 步驟三：硬體設備安裝
### 1.鏡頭連接<br />
參考:[Getting started with the Camera Module](https://projects.raspberrypi.org/en/projects/getting-started-with-picamera)<br />
### 2.SW-420震動感測模組連接<br />
參考教學:[RASPBERRY PI 3 MODEL 與 SW-420 震動感測模組](https://blog.everlearn.tw/%E7%95%B6-python-%E9%81%87%E4%B8%8A-raspberry-pi/raspberry-pi-3-model-%E8%88%87%E9%9C%87%E5%8B%95%E6%84%9F%E6%B8%AC%E5%99%A8-sw-420)<br />
共有三隻接腳，分別是工作電源 (Vcc)、接地與數位輸出，在電路板上都有明確標示。<br />
  (1)SW-420 可使用 +3V 的電壓，因此工作電源 (Vcc)接腳連結至 Raspberry Pi 實體編號 1 的腳位。<br />
  (2)SW-420 的接地接腳連結至 Raspberry Pi 實體編號 6 的腳位 (接地)。<br />
  (3)SW-420 的數位輸出腳位連結至 Raspberry Pi 實體編號 12 的腳位，其 BCM 編號為 18。<br />
[參考圖片](https://blog.everlearn.tw/wp-content/uploads/2017/11/raspberry_pi_sw420_bb.png)
# 步驟四：coding python程式碼<br />
### 1.創建一個資料夾<br />
拿來放剛才下載下來的憑證json檔，以及等下的python黨及結果。(在這裡我創建在`Desktop`上，並叫做`python_text`<br />
### 2.Google API程式碼部分(這部分只有單純的放圖片並辨識文字結果而已)<br />
參考[Google Developers 官方文件](https://developers.google.com/drive/api/v3/quickstart/python)
    #!/usr/bin/python3
    from __future__ import print_function
    import httplib2
    import os
    import io
    
    from apiclient import discovery
    from oauth2client import client
    from oauth2client import tools
    from oauth2client.file import Storage
    from apiclient.http import MediaFileUpload, MediaIoBaseDownload

    try:
      import argparse
      flags = argparse.ArgumentParser(parents=[tools.argparser]).parse_args()
    except ImportError:
      flags = None

    SCOPES = 'https://www.googleapis.com/auth/drive'
    CLIENT_SECRET_FILE = 'client_id.json'
    APPLICATION_NAME = 'Python OCR'

    def get_credentials():
      """取得有效的憑證
         若沒有憑證，或是已儲存的憑證無效，就會自動取得新憑證

         傳回值：取得的憑證
      """
      credential_path = os.path.join("./", 'google-ocr-credential.json')
      store = Storage(credential_path)
      credentials = store.get()
      if not credentials or credentials.invalid:
        flow = client.flow_from_clientsecrets(CLIENT_SECRET_FILE, SCOPES)
        flow.user_agent = APPLICATION_NAME
        if flags:
          credentials = tools.run_flow(flow, store, flags)
        else: # Needed only for compatibility with Python 2.6
          credentials = tools.run(flow, store)
        print('憑證儲存於：' + credential_path)
      return credentials

    def main():

      # 取得憑證、認證、建立 Google 雲端硬碟 API 服務物件
      credentials = get_credentials()
      http = credentials.authorize(httplib2.Http())
      service = discovery.build('drive', 'v3', http=http)

      # 包含文字內容的圖片檔案（png、jpg、bmp、gif、pdf）
      imgfile = 'sample.jpg'

      # 輸出辨識結果的文字檔案
      txtfile = 'output.txt'

      # 上傳成 Google 文件檔，讓 Google 雲端硬碟自動辨識文字
      mime = 'application/vnd.google-apps.document'
      res = service.files().create(
        body={
          'name': imgfile,
          'mimeType': mime
        },
        media_body=MediaFileUpload(imgfile, mimetype=mime, resumable=True)
      ).execute()

      # 下載辨識結果，儲存為文字檔案
      downloader = MediaIoBaseDownload(
        io.FileIO(txtfile, 'wb'),
        service.files().export_media(fileId=res['id'], mimeType="text/plain")
      )
      done = False
      while done is False:
        status, done = downloader.next_chunk()

    if __name__ == '__main__':
      main()
      
### 3.拍照程式碼部分<br />
    from picamera import PiCamera
    from time import sleep

    camera = PiCamera()

    camera.start_preview()
    sleep(2)
    camera.capture('/home/pi/Desktop/pytohn_text/image.jpg')
    camera.stop_preview()
### 4.錄影程式碼部分(這裡指的是單純十秒的鏡頭錄影 實際合併後的程式碼錄影結束時間為接收到碰撞的當下)<br />
    from picamera import PiCamera
    from time import sleep

    camera = PiCamera()

    camera.start_preview()
    camera.start_recording('/home/pi/Desktop/video.mp4')
    sleep(10)
    camera.stop_recording()
    camera.stop_preview()
### 5.合併所有部分，並存成python檔，放入python_text資料夾中(這裡取名為google_api.py)<br />
    #!/usr/bin/python3
    from __future__ import print_function
    import httplib2
    import os
    import io
    import time
    import RPi.GPIO as GPIO

    from apiclient import discovery
    from oauth2client import client
    from oauth2client import tools
    from oauth2client.file import Storage
    from apiclient.http import MediaFileUpload, MediaIoBaseDownload
    from picamera import PiCamera
    from time import sleep

    try:
      import argparse
      flags = argparse.ArgumentParser(parents=[tools.argparser]).parse_args()
    except ImportError:
      flags = None

    SCOPES = 'https://www.googleapis.com/auth/drive'
    CLIENT_SECRET_FILE = 'client_id.json'
    APPLICATION_NAME = 'Python OCR'
    
    # 執行錄影、拍照及API主程式的函式
    def camera():
        camera = PiCamera()
        camera.start_recording('/home/pi/Desktop/python_text/video.h264')
        while True:
            if GPIO.input(SW420_PIN) == GPIO.HIGH: 
                my_callback(SW420_PIN)
                camera.stop_recording()
                camera.stop_preview()
                sleep(2)
                camera.start_preview()
                sleep(2)
                camera.capture('/home/pi/Desktop/python_text/image.jpg')
                camera.stop_preview()
                main()
                time.sleep(5)
                break;

    def get_credentials():
      """取得有效的憑證
         若沒有憑證，或是已儲存的憑證無效，就會自動取得新憑證

         傳回值：取得的憑證
      """
      credential_path = os.path.join("./", 'google-ocr-credential.json')
      store = Storage(credential_path)
      credentials = store.get()
      if not credentials or credentials.invalid:
        flow = client.flow_from_clientsecrets(CLIENT_SECRET_FILE, SCOPES)
        flow.user_agent = APPLICATION_NAME
        if flags:
          credentials = tools.run_flow(flow, store, flags)
        else: # Needed only for compatibility with Python 2.6
          credentials = tools.run(flow, store)
        print('憑證儲存於：' + credential_path)
      return credentials

    def main():

      # 取得憑證、認證、建立 Google 雲端硬碟 API 服務物件
      credentials = get_credentials()
      http = credentials.authorize(httplib2.Http())
      service = discovery.build('drive', 'v3', http=http)

      # 包含文字內容的圖片檔案（png、jpg、bmp、gif、pdf）
      imgfile = 'image.jpg'

      # 輸出辨識結果的文字檔案
      txtfile = 'output.txt'

      # 上傳成 Google 文件檔，讓 Google 雲端硬碟自動辨識文字
      mime = 'application/vnd.google-apps.document'
      res = service.files().create(
        body={
          'name': imgfile,
          'mimeType': mime
        },
        media_body=MediaFileUpload(imgfile, mimetype=mime, resumable=True)
      ).execute()

      # 下載辨識結果，儲存為文字檔案
      downloader = MediaIoBaseDownload(
        io.FileIO(txtfile, 'wb'),
        service.files().export_media(fileId=res['id'], mimeType="text/plain")
      )
      done = False
      while done is False:
        status, done = downloader.next_chunk()

      # 刪除剛剛上傳的 Google 文件檔案
      #service.files().delete(fileId=res['id']).execute()
    SW420_PIN = 18
    LED_PIN = 23

    def my_callback(channel):
        print('感受震動')
        GPIO.output(LED_PIN, GPIO.HIGH)
        time.sleep(0.1)
        GPIO.output(LED_PIN, GPIO.LOW)

    GPIO.setmode(GPIO.BCM)
    GPIO.setup(SW420_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)
    GPIO.setup(LED_PIN, GPIO.OUT)

    if __name__ == '__main__':
        try:
            camera()
        except KeyboardInterrupt:
            print('關閉程式')
        finally:
            GPIO.cleanup()
# 步驟五：執行程式
### 1.進入console執行程式<br />

    $cd Desktop
    $cd python_text
    $python3 google_api.py
### 2.觸發震動感測器
過了幾秒後，你就能在你的google雲端上看見撞擊當下的照片以及車牌號碼，並能在樹莓派中查看撞擊前的錄影畫面。
