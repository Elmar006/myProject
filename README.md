package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
	"time"

	"gopkg.in/telebot.v3"
)

const (
	coinGeckoAPI = "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum,solana&vs_currencies=usd"
)

type CoinPrices struct {
	Bitcoin  map[string]float64 `json:"bitcoin"`
	Ethereum map[string]float64 `json:"ethereum"`
	Solana   map[string]float64 `json:"solana"`
}

func main() {
	pref := telebot.Settings{
		Token:  "7373716048:AAFQ3zPobnK1DfpHB-eiScZo_V3JDQweapg",
		Poller: &telebot.LongPoller{Timeout: 10 * time.Second},
	}

	bot, err := telebot.NewBot(pref)
	if err != nil {
		panic(err)
	}

	bot.Handle("/start", func(c telebot.Context) error {
		return c.Send("Привет! Я крипто-бот. Используй команду /rates чтобы узнать текущие курсы")
	})

	bot.Handle("/rates", func(c telebot.Context) error {
		prices, err := getCryptoPrices()
		if err != nil {
			return c.Send("Не могу получить данные. Попробуйте позже.")
		}

		message := fmt.Sprintf(
			"📊 Текущие курсы:\n\n"+
				"₿ Bitcoin: $%.2f\n"+
				"Ξ Ethereum: $%.2f\n"+
				"◎ Solana: $%.2f",
			prices.Bitcoin["usd"],
			prices.Ethereum["usd"],
			prices.Solana["usd"],
		)

		return c.Send(message)
	})

	bot.Start()
}

func getCryptoPrices() (*CoinPrices, error) {
	resp, err := http.Get(coinGeckoAPI)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return nil, err
	}

	var prices CoinPrices
	if err := json.Unmarshal(body, &prices); err != nil {
		return nil, err
	}

	return &prices, nil
}


