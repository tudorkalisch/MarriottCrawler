#MarriottDeepCrawler


This is the final stage of my first Crawler. With this Crawler I want to get Name,Score,Url, Phone Number, Address and Number of Reviews for each Hotel. I used the Scrapy framework to implement and develop this project. With this Crawler I want to get Name,Score,Url, Phone Number, Address and Number of Reviews for each Hotel. With this project I learned a lot about How I should parse trough webpages,Requests/Responses,HTML structure and Regular Expressions.


This project went trough 3 stages, first when I tried to do this without Rules which did not quite work because I couldn't get all the information about each Hotel, only partial information.

Second stage was when I got to Rules and LinkExtractors when the project was working and is actually the version that is currently here written in the MarriotCrawler.py.

Third when I created a Scrapy Project instead of the simple Python Flie because I learned that the scrapy project can have multiple settings and spiders in the same project and it can be easily modified and updated for further use.

The spider is located in MarriottCrawler/marriottspider/marriottspider/spiders/MarriottCrawler.py

**Initial Page** http://www.marriott.com/hotel-search/

#Documentation
#Class attributes:

**name** =’marriott’ → name of the spider

**allowed_domains** =[ ‘marriott.com’] → with this we stop the spider from crawling to other pages

**start_urls** = [ http://www.marriott.com/hotel-search/’ ]→ the starting page from which the spider crawls

------------------------------------------------------------------------------------------------------------

**rules** → with rules and link extractors we define how the links will be extracted from each crawled page
E.g.:

-First we search for links like:
http://www.marriott.com/hotel-search/south-africa.hotels/
http://www.marriott.com/hotel-search/arkansas.hotels.united-states/
http://www.marriott.com/hotel-search/north-dakota.hotels.united-states/
and we use the first rule to extract them

	Rule(LinkExtractor(allow=('http:\/\/www\.marriott\.com\/hotel-search\/[A-Za-z\-]+[\.]+hotels+([\.]united-states[\/]|[\/])', )))

-Second we search for links like:
www.marriott.com/hotel-search/arkansas.hotels.united-states.page2/
http://www.marriott.com/hotel-search/south-africa.hotels.page4/
http://www.marriott.com/hotel-search/thailand.hotels.page2/
and we use the second rule to extract them

	Rule(LinkExtractor(allow=('http:\/\/www\.marriott\.com\/hotel-search\/[A-Za-z\-]+[\.]+hotels+([\.]united-states[\.]+page+[0-9]+[\/]|[\.]+page+[0-9]+[\/])')))

-Finally we  search for links like:
http://www.marriott.com/hotels/travel/hktkl-jw-marriott-khao-lak-resort-and-spa/
http://www.marriott.com/hotels/travel/durka-protea-hotel-karridene-beach/
http://www.marriott.com/hotels/travel/rogri-residence-inn-bentonville-rogers/
using the final rule we also call the parse_item function which gets the information about the hotel and writes it in the .csv file

	Rule(LinkExtractor(allow=('http:\/\/www\.marriott\.com\/hotels\/travel\/[a-z0-9A-Z\-\.]+[^w]')),callback='parse_item')
    

------------------------------------------------------------------------------------------------------------

**parse_item**:

-Desc: We are using responses and selectors to extract the information that we need
about the hotel and write it into the .csv file using the scrapy crawl option
**scrapy crawl marriott -o filename.csv -t csv**

    def parse_item(self, response):
    
        yield {
            'name': response.css("a.t-inherit-styles.analytics-click span::text").extract_first().encode('utf-8'),
            'score': response.css("div.l-rating-normal-outof span::text").extract_first(),
            'url': "www.marriott.com" + response.css("a.t-inherit-styles.analytics-click::attr('href')").extract_first().encode('utf-8'),
            'address': self.createAddress(response.css("a.t-address.is-cursor-pointer.analytics-click span::text").extract()).encode('utf-8'),
            'phone': response.css("span.is-hiddenPhone-Number::text").extract_first(),
            'number ratings': self.createRatings(response.css("div.l-float-left.l-margin-left-quarter span::text").re("\d+"))
        }
E.g:
'score': response.css("div.l-rating-normal-outof span::text").extract_first()
extracts the score of the Hotel
Note:address and the number of ratings are different and we use unique functions to get the information that we need

	def createAddress(self,input_list):
    s = ""
    for item in input_list:
        s += str(item) + ','
    return s
    
	def createRatings(self,input_review):
    if len(input_review) >= 1:
        return input_review[0]
    return ""
