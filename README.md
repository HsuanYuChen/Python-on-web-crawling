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
    from lxml import etree  只從 lxml <模組module>，裝備 etree <物件Object>
    page = etree.HTML(html)
    把名稱為html的資料(網頁抓下來的原始碼)，
    以 物件etree的HTML函式，轉換成「XPath的節點(node)型態」，
            並以名稱page紀錄。

      import requests
      url = "網址"
      response = requests.get(url)
      html = response.text
      
      from lxml import etree
      page = etree.HTML(html)
      first_article_tags = page.xpath("//footer/a/text()")[0]
      print (first_article_tags)
      for article_title in page.xpath("//h5/a/text()"):
      print (article_title)
