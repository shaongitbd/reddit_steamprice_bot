import praw
import requests
from currency_converter import CurrencyConverter
import re

from bs4 import BeautifulSoup


def price_scraper(message):
        game_region = ['AR', 'TR', 'IN', "BR", "kz"]
        game_country = ["Argentina", "Turkey", "India", "Brazil", "Kazakhstan"]
        currency = ["ARS", "TRY", "INR", "BRL", "KZT"]
        game_prices = []
        converted_price = []
        message = message.replace("gamedeals", "")
        message = message.replace("price", "")

        search_page = requests.get("https://store.steampowered.com/search/?term=" + message)
        soup = BeautifulSoup(search_page.text, 'html.parser')
        game_page = soup.find("a", class_="search_result_row ds_collapse_flag")
        game_url = game_page["href"].split("?")[0]

        game_name = soup.find("span", class_="title").text
        game_release = soup.find("div", class_="col search_released responsive_secondrow").text
        game_original_price = soup.find("div", class_="col search_price responsive_secondrow").text
        game_original_price = ''.join(x for x in game_original_price if x.isdigit() or x == "," or x == ".")


        for i in game_region:
            single_page = requests.get(game_url + "?cc=" + i)
            soup = BeautifulSoup(single_page.text, "html.parser")
            regional_price = soup.find("div", class_="game_purchase_price").text
            regional_price = ''.join(x for x in regional_price if x.isdigit() or x == "," or x == ".")

            regional_price = regional_price.replace(",", ".")

            game_prices.append(regional_price)

        if int(len(str(regional_price))) > 0:
            for i in game_prices:
                curr_api = "https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=" + \
                           currency[
                               game_prices.index(i)] + "&to_currency=USD&apikey=yourapi"
                curr_req = requests.get(curr_api).json()

                rate = curr_req['Realtime Currency Exchange Rate']['5. Exchange Rate']
                converted_price.append(float(rate) * float(i))

            post = ""
            post += "Game Name: " + game_name + "\n\n"
            post += "Released :" + game_release + "\n\n"
            post += game_original_price + " original price" + "\n\n"
            for j in converted_price:
                post += "Region:" + game_country[converted_price.index(j)] + "(" + game_region[
                    converted_price.index(j)] + ")" + " Price:" + str(
                    game_prices[converted_price.index(j)]) + " Converted: $" + "{:.2f}".format(j) + "\n\n"

        return post


r = praw.Reddit(
    username="your username",
    password="Your password",
    client_id = "clientid",
    client_secret = "clientsecret",
    user_agent = "<console:GameDeals:1.0>")


messages = r.inbox.stream()
print(messages)

for message in messages:
  try:
    if message in r.inbox.mentions() and message in r.inbox.unread():
        print(message.body)
        if 'price' in message.body:

            reddit_msg = price_scraper(message.body)


        else:
            reddit_msg="Hey there, let's learn how to use me. \n\n To get cheapest game prices in different regions type: !igamedeals price nameofgame"

        message.reply(reddit_msg)
        message.mark_read()

  except praw.exceptions.APIException:
    print("Rate Limited..Please Try again later")



