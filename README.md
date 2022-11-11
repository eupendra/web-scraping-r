# Web Scraping With R

- [Installing requirements](#installing-requirements)
- [Web scraping with rvest](#web-scraping-with-rvest)
- [Web scraping with RSelenium](#web-scraping-with-rselenium)


## Installing requirements

For macOS, run the following:

```shell
brew install r
brew install --cask r-studio

```

For Windows, run the following:

```batch
choco install r.project
choco install r.studio
```

### Installing required libraries

```R
install.packages("rvest")
install.packages("dplyr")
```

## Web scraping with rvest

```R
library(rvest)
link = "https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes"
page = read_html(link)

```

### Parsing HTML Content

```R
page %>% html_elements(css="")
page %>% html_elements(xpath="")
```



![](https://oxylabs.io/blog/images/2021/12/wiki_markup.png)

For above page, use the following:

```R
htmlElement <- page %>% html_element("table.sortable")
```

### Saving data to a data frame

```R
df <- html_table(htmlEl, header = FALSE)
names(df) <- df[2,]
df = df[-1:-2,]
```

### Exporting data frame to a CSV file

```R
write.csv(df, "iso_codes.csv")
```

### Downloading Images

```R
page <- read_html(url)
image_element <- page %>% html_element(".thumbborder")
image_url <- image_element %>% html_attr("src")
download.file(image_url, destfile = basename("paris.jpg"))
```

### Scrape Dynamic Pages with Rvest

Find the API endpoint and use that as following:
```R
page<-read_html(GET(api_url, timeout(10)))
jsontext <- page %>% html_element("p")  %>% html_text()
```
For a complete example, see [dynamic_rvest.R](src/dynamic_rvest.R).

## Web scraping with RSelenium

```R
install.package("RSelenium")
library(RSelenium)

```

### Starting Selenium

#### Method 1

```R
# Method 1
rD <- rsDriver(browser="chrome", port=9515L, verbose=FALSE)
remDr <- rD[["client"]]

```

#### Method 2

```shell
docker run -d -p 4445:4444 selenium/standalone-firefox
```

```R
remDr <- remoteDriver(
  remoteServerAddr = "localhost",
  port = 4445L,
  browserName = "firefox"
)
remDr$open()
```

### Working with elements in Selenium

```R
remDr$navigate("https://books.toscrape.com/catalogue/category/books/science-fiction_16")
```

![](https://oxylabs.io/blog/images/2021/12/book_title.png)

```R
titleElements <- remDr$findElements(using = "xpath", "//article//img")
titles <- sapply(titleElements, function(x){x$getElementAttribute("alt")[[1]]})

pricesElements <- remDr$findElements(using = "xpath", "//*[@class='price_color']")
prices <-  sapply(pricesElements, function(x){x$getElementText()[[1]]})

stockElements <- remDr$findElements(using = "xpath", "//*[@class='instock availability']")
stocks <-  sapply(stockElements, function(x){x$getElementText()[[1]]})

```

### Creating a data frame

```R
df <- data.frame(titles, prices, stocks)
```

#### Save CSV

```R
write.csv(df, "books.csv")
```
