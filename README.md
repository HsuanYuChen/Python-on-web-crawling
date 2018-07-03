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
- 基礎1 : 抓取圖片

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
- 基礎2 : 抓取網頁資料

    from lxml import etree 
    
    只從 lxml <模組module>，裝備 etree <物件Object>
    
    page = etree.HTML(html)
    
    把名稱為html的資料(網頁抓下來的原始碼)，
    以 物件etree的HTML函式，轉換成「XPath的節點(node)型態」，並以名稱page紀錄。
    
    XPath 節點(node)選擇：
    
     1. 小孩/：「下一層節點」，或是該標籤的「屬性 @」或「文字 text()」
     2. 子孫//：全搜索，常用在搜尋不知道有幾層節點的狀況
     
           放在開頭，就是整份文件搜尋：
           
           //table 是找尋全文件中的table標籤(整份文件的子孫)
           
           放在中間，就是前一個節點node(父節點)下全搜尋：
           
           //table//text() 是找尋全文件中table標籤底下的所有文字(table下的子孫)
     3. . 以現在的節點node搜索，常用在同時呈現同一階層(輩份)的資料
    

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
-
