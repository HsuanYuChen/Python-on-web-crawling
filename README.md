# Python 爬蟲經驗分享
可用套件：urllib、requests、lxml、beautiful_soup、scrapy、selenium
目的：
幫我們自動化從網頁上去擷取我們所需要的資料，減少重複的動作
1. 點擊網頁 or 輸入查詢資料
2. 等待網頁開啟
3. 選擇「想要」的內容
4. 存到另外的地方
5. 重覆以上動作

# RoadMap
- 基礎1 : Request Method

      GET
      url = 網址
      response = requests.get(url)
      1. 缺點就是資料會被看光
      2. 內容跟地址字數就會受到大小限制
      
      POST
      url = http://www.thsrc.com.tw/tw/TimeTable/SearchResult
      response = requests.post(url,data=post_data)
      1. 內容是被保護起來
      2. 只看得到地址，看不到內容，安全性會提高
      3. 容量比使用get高很多
# 
- 範例：抓取圖片

      import requests (使用 requests 模組，來抓取以傳遞資料的方法為 GET 的網頁原始碼)
      url = "任何網站"
      response = requests.get(url)
      html = response.text

      #擷取屬於圖片的網址
      for temp in html.split("<img"): 
          line = temp.split("/>")[0]
          if ("src=" in line):
             img_src = line.replace("\'","\"").split("src=\"")[-1].split("\"")[0]
             if ( (".jpeg" in img_src) or (".jpg" in img_src) or (".JPG" in img_src) or (".png" in img_src) ) :
             # 抓取圖片
                 img_response = requests.get(img_src)
                 img = img_response.content  #因為編碼問題，使用response.content
            
                 filename=img_src.split("/")[-1]
                 filepath="tmp/"+filename
                 fileout = open(filepath,"wb")
                 fileout.write(img)
- 基礎2 : 使用XPath
- 好用 Chrome 擴充功能

     [Quick Javascript Switcher](https://chrome.google.com/webstore/detail/quick-javascript-switcher/geddoclleiomckbhadiaipdggiiccfje) ：快速知道網頁哪些內容是由JavaScript(JS)產生
      
     [XPath Helper](https://chrome.google.com/webstore/detail/xpath-helper/hgimnogjllphhhkhlmebbmlgjoejdpjl?hl=zh-TW)：了解XPath語法會抓到哪些內容

            1. from lxml import etree    
               只從 lxml <模組module>，裝備 etree <物件Object>    
            2. page = etree.HTML(html)
               把名稱為html的資料(網頁抓下來的原始碼)，
               以 物件etree的HTML函式，轉換成「XPath的節點(node)型態」，並以名稱page紀錄。  
    
            3. XPath 節點(node)選擇：
    
                   1. 小孩/：「下一層節點」，或是該標籤的「屬性 @」或「文字 text()」
                   2. 子孫//：全搜索，常用在搜尋不知道有幾層節點的狀況
                        放在開頭，就是整份文件搜尋：
                              //table 是找尋全文件中的table標籤
                        放在中間，就是前一個節點node(父節點)下全搜尋：
                              //table//text() 是找尋全文件中table標籤底下的所有文字
                   3. . 以現在的節點node搜索，常用在同時呈現同一階層(輩份)的資料
                   
            4. XPath 敘述(Predicates):
            
                   1. 使用[]表示
                   2. 位置：
                        div[1]：標籤div的第1個node(從1開始數)
                   3. 屬性狀態：
                        header[@class='entry-header']：header標籤中屬性class為entry-header的node
                   4. 屬性存在與否: 
                        li[@class]：標籤li中有class這個屬性的node
                        li[not(@class)]：標籤li中「沒有」class這個屬性的node
                   5. contains(屬性,子字串)
                        同等於Python的in，比較屬性的值是否包含子字串
                        eg. div[@class='post-body entry-content'] 可以改寫成 div[contains(@class, 'post-body')]
                   6. or/and
                        同等於Python的or/and，邏輯中「或/和」的概念
                        eg. font[(@color="#0000ff" or @color="blue")]
                   7. XPath 的串接
                        | ：可以串接兩個Path
                        常用情境：同時抓取位於不同Path的所需資訊：第一頁出現的文章標題以及熱門文章標題
                   8. [更多 XPath 語法](https://www.w3schools.com/xml/xpath_syntax.asp)                       
# 
- 範例：抓取網頁資料
            import requests
            url = "網址"
            response = requests.get(url)
            html = response.text
      
            from lxml import etree
            page = etree.HTML(html)
            first_article_tags = page.xpath("//footer/a/text()")[0] #Python的語法，所以[0]是取「第1個」結果。
            print (first_article_tags)
            for article_title in page.xpath("//h5/a/text()"):
            print (article_title)
            #xpath找尋
            整個文件中的子孫 // ， 所有是 footer 標籤，
            且小孩是 a 標籤的文字 text()的結果(為一串列)，
            而我們只取「第1個」結果，存到變數 first_article_tags 中。
            
            #抓取網頁上的圖片
            from lxml import etree
            import requests

            url = "網址"
            response = requests.get(url)
            html = response.text

            page = etree.HTML(html)

            for img_src in page.xpath("//img/@src"):
            # 抓取圖片
            img_response = requests.get(img_src)
            img = img_response.content
            
            注意：圖片的內容不是文字！        
            所以用的是 content ，而不是像之前抓網頁的時候是用 text ，是取得bytes型式的內容        
            寫檔的時候，因為取得的是bytes型式的內容，因此用的是 wb ，而不是 w

            filename=img_src.split("/")[-1]
            filepath="tmp/"+filename
            fileout = open(filepath,"wb")
            fileout.write(img)
            fileout.close()
- 基礎3: Cookie介紹

      Cookie
      requests.get(url,cookies=我的cookie資訊)
      requests.post(url,data=post_data,cookies=我的cookie資訊)
      1. 資料記錄在Client上
      2. 紀錄的內容：登入狀態(使用者名稱)、瀏覽紀錄......etc.
      3. 應用
      記住我的帳號
      購物車在付帳前要紀錄使用者購買哪些商品
      推薦你可能喜歡的商品
     範例： 參考[斧頭幫大挑戰第三關](https://github.com/HsuanYuChen/Python-on-web-crawling/blob/master/斧頭幫大挑戰第三關)
     應用於網頁分頁透過 &page={prev,next} 這兩種值來決定要上一頁或者下一頁
     
      Session
      資料記在Server上
      存放較敏感的資料 
      
      Cache
      加速瀏覽速度（如：有時候網路斷掉，Youtube還能繼續播放一段時間; 第二次開網頁比較快; 網站更新版本需要再特別refresh）
      圖檔、JavaScript、Xml......etc
