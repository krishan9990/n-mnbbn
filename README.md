from playwright.async_api import async_playwright
import asyncio
import re
import pandas as pd
# from parsing_tools import BeautifulSoup
# from parsing_tools import ParsingTools
from urllib.request import urlopen
from bs4 import BeautifulSoup

All_data = []


def extract_data(All_data):
    for i in All_data:
        Bank = re.findall('projectTable_bankBean.bankName"[^\"]+"[^=]+=[^=]+=.([^"]+)', str(i))
        Branch = re.findall('aria-describedby[^"]+"projectTable_branchBean.branchName[^=]+=[^=]+=[^=]+="([^"]+)"',
                            str(i))
        Quater = re.findall('projectTable_quarterBean.quarterDateStr"[^"]+"[^=]+=[^=]+=.([^"]+)', str(i))
        Address = re.findall('projectTable_importDataBean.regaddr"[^"]+"[^=]+=[^=]+=.([^"]+)', str(i))
        Direct_name = re.findall('projectTable_directorName"[^"]+"[^=]+=[^=]+=.([^"]+)', str(i))
        Amount = re.findall('projectTable_totalAmount"[^"]+"[^=]+=[^=]+=.([^"]+)', str(i))
        Borrowser_name = re.findall('projectTable_borrowerName"[^"]+"[^=]+=[^=]+=.([^"]+)', str(i))

        df = pd.DataFrame(
            {'Bank': Bank, 'Branch': Branch, 'Quater': Quater, 'Borrowser_name': Borrowser_name, 'Address': Address,
             'Direct_name': Direct_name, 'Amount': Amount})
        # print(df)
        df.to_csv('file1.csv')

        # print(Bank)

        # print('           ')
        # print(Branch)
        # print('           ')
        # print(Quater)
        # print('           ')
        # print(Borrowser_name)
        # print('           ')
        # print(Address)
        # print('           ')
        # print(Direct_name)
        # print('           ')
        # print(Amount)


async def main():
    async with async_playwright() as pw:
        browser = await pw.chromium.launch(
            headless=False  # Show the browser
        )
        page = await browser.new_page()
        await page.goto('https://opensea.io/explore-collections')
        # Data Extraction Code Here
        await page.evaluate('window.scrollBy(0,9000)')
        print("enter step1")
        for i in range(1, 2):
            html_data = await page.content()
            await page.evaluate('window.scrollBy(0,900)')
            soup = BeautifulSoup(html_data, "html5lib")
            list_div = soup.findAll("table", {"id": "projectTable"})
            print(list_div,"---------------------")
            All_data.append(list_div)
            # print(All_data)
            # await page.click('//td[@id = "next_pagingDiv"]')
            # await page.wait_for_timeout(40000)
            # print(i)
        # await page.evaluate('window.scrollBy(0,500)')
        # print(All_data)
        print(len(All_data))
        extract_data(All_data)
        await page.wait_for_timeout(600000)
        await browser.close()


if __name__ == '__main__':
    asyncio.run(main())
