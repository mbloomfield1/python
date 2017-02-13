# python
#python projects

import csv


class AbstractRecord(object):
    def __init__(self, name):
        self.name = name
        self.data = []


class StockStatRecord(AbstractRecord):

    def __init__(self, ticker, exchange_country, price, exchange_rate,
                 shares_outstanding, net_income, market_value_usd, pe_ratio):
        self.name = ticker
        self.data = [exchange_country, price, exchange_rate,
                     shares_outstanding, net_income, market_value_usd, pe_ratio]

    def __str__(self):
        s = '<Stock> ({0}, {1}, {2:.2f}, {3:.2f}, {4:.2f}, {5:.2f}, {6:.2f}, {7:.2f})'
        return s.format(self.name, *self.data)


class BaseballStatRecord(AbstractRecord):

    def __init__(self, name, salary, G, AVG):
        self.name = name
        self.data = [salary, G, AVG]

    def __str__(self):
        s = '<Baseball> ({0}, {1}, {2}, {3:.2f})'
        return s.format(self.name, *self.data)


class BadData(Exception):
    pass


class AbstractCSVReader(object):

    def __init__(self, file_path):
        self.file_path = file_path

    def row_to_record(self, row):
        raise NotImplementedError

    def load(self):
        rows = []
        with open(self.file_path) as f:
            for d in csv.DictReader(f):
                try:
                    row = self.row_to_record(d)
                except BadData:
                    continue
                rows.append(row)
        return rows


class StockCSVReader(AbstractCSVReader):

    def row_to_record(self, row):
        ticker = row['ticker']
        exchange_country = row['exchange_country']
        price = row['price']
        exchange_rate = row['exchange_rate']
        shares_outstanding = row['shares_outstanding']
        net_income = row['net_income']

        for ele in [ticker, exchange_country, price, exchange_rate,
                    shares_outstanding, net_income]:
            if ele == '':
                raise BadData

        try:
            ticker = int(ticker)
            price = float(price)
            exchange_rate = float(exchange_rate)
            shares_outstanding = float(shares_outstanding)
            net_income = float(net_income)
        except ValueError:
            raise BadData

        market_value_usd = price * exchange_rate * shares_outstanding
        try:
            pe_ratio = price / float(net_income)
        except ZeroDivisionError:
            raise BadData

        return StockStatRecord(ticker, exchange_country, price, exchange_rate,
                               shares_outstanding, net_income,
                               market_value_usd, pe_ratio)


class BaseballCSVReader(AbstractCSVReader):

    def row_to_record(self, row):
        player_name = row['PLAYER']
        games = row['G']
        avg = row['AVG']
        salary = row['SALARY']

        for ele in [player_name, games, avg, salary]:
            if ele == '':
                raise BadData

        try:
            games = int(games)
            avg = float(avg)
            salary = int(salary)
        except ValueError:
            raise BadData

        return BaseballStatRecord(player_name, salary, games, avg)


if __name__ == '__main__':
    print('Baseball Data\n')
    print('Player name: Salary: Games: AVG:')

    baseball = BaseballCSVReader('MLB2008.csv').load()
    for row in baseball:
        print(row)


    print('\n\nStock Data')
    stocks = StockCSVReader('StockValuations.csv').load()
    print('Ticker, Exchange Country, Price, Exchange Rate, Shares Outstanding, Net Income, Market Value, PE Ratio')

    for row in stocks:
        print(row)
