# Trade Product Data Scraper

A pair of web scraping applications designed to extract structured product information from trade websites.

This project contains two closely related applications: one for **Bachmann Trade** and one for **Gaugemaster Trade**. Both applications retrieve product information directly from the websites' HTML and convert the relevant data into a more usable format.

The project was built to automate the collection of product information that would otherwise need to be manually gathered from individual product pages.

## Features

* Extracts product information directly from website HTML
* Supports both Bachmann Trade and Gaugemaster Trade
* Automatically processes product pages
* Converts web page data into structured information
* Standalone Windows executables are included for ease of use
* Separate implementations allow each scraper to account for differences in website structure

## Applications

### Bachmann Product Extractor

Located in:

```text
BM Product Extractor/
```

Standalone executable:

```text
Bachmann Product Extractor.exe
```

This application extracts product data from the Bachmann Trade website.

### Gaugemaster Product Extractor

Located in:

```text
GM Product Extractor/
```

Standalone executable:

```text
Gaugemaster Product Extractor.exe
```

This application performs the same general task against the Gaugemaster Trade website.

Although the applications have similar functionality, they use separate implementations because the websites have different HTML structures and therefore require different approaches to locating and extracting the required information.

## How It Works

The applications follow a simple scraping pipeline:

```text
Website
   ↓
Retrieve HTML
   ↓
Parse HTML
   ↓
Locate Product Information
   ↓
Extract Relevant Data
   ↓
Process / Format Data
   ↓
Output
```

Rather than relying on a fixed API, the applications work with the HTML returned by the websites and identify the relevant elements containing product information.

This required analysing the structure of each website and developing extraction logic specific to each site's layout.

## Why Two Extractors?

The Bachmann and Gaugemaster websites provide similar types of product information, but their HTML structures are different.

Instead of attempting to force both websites through a single scraper, the project separates the extraction logic into two applications.

This makes the code easier to reason about and allows changes to one website's structure to be handled without unnecessarily affecting the other implementation.

## Technologies

* **Python**
* **HTML parsing**
* **Web scraping**
* **Data processing**
* **Windows executable packaging**

## Project Structure

```text
Trade-Product-Data-Scraper/
│
├── BM Product Extractor/
│   └── Bachmann Product Extractor.exe
│   └── Bachmann Product Extractor.py
│
├── GM Product Extractor/
│   └── Gaugemaster Product Extractor.exe
│   └── Gaugemaster Product Extractor.py
│
└── README.md
```

## Running the Applications

### Using the executables

Windows users can run the relevant `.exe` file directly:

```text
Bachmann Product Extractor.exe
```

or:

```text
Gaugemaster Product Extractor.exe
```

### Running from source

The source code for each extractor is available in its respective directory.

Open the relevant project directory and run the application's Python entry point.

## Development

This project was developed as a practical exercise in automating data collection from real-world websites.

One of the main challenges was that the required information was contained within the websites' HTML rather than being provided through a dedicated API. This meant identifying the relevant HTML elements and writing extraction logic capable of retrieving the required information reliably.

Working with two different websites also provided experience in adapting a common concept to different underlying implementations.

## Future Improvements

Potential improvements include:

* Combining both extractors behind a common interface
* Adding additional trade websites
* Improving error handling for unavailable or changed pages
* Adding logging for failed extractions
* Adding automated validation of extracted data
* Making the output format configurable
* Adding automated tests for the extraction logic
* Improving resilience against changes to website HTML structures

## Disclaimer

This project is intended for personal and educational use.

The applications retrieve accessible webpage data and are not intended to bypass authentication, access restricted information, or interfere with the operation of the websites being accessed. An account may be required to access the respective websites.

---

**Author:** Joshua Cheetham
