# Advanced-Financial-Modeling
Cody Heiland Personal Project
import yahoo_fin.stock_info as si
from datetime import datetime, timedelta
from flask import Flask, render_template_string
import plotly.graph_objects as go
import plotly.offline as pyo

app = Flask(__name__)

def get_stock_info(symbol):
    try:
        info = si.get_quote_table(symbol)
        beta = info.get('Beta (5Y Monthly)', 'N/A')
        market_cap = info.get('Market Cap', 'N/A')
        dividend_yield = info.get('Forward Dividend & Yield', 'N/A')
        return beta, market_cap, dividend_yield
    except Exception as e:
        print(f"Error fetching data for {symbol}: {e}")
        return 'N/A', 'N/A', 'N/A'
def get_monthly_change(symbol):
    first_of_month = datetime.now().replace(day=1)
    try:
        monthly_data = si.get_data(symbol, start_date=first_of_month.date())
        if not monthly_data.empty:
            month_open = monthly_data['open'][0]
            month_close = monthly_data['close'][-1]
            return ((month_close - month_open) / month_open) * 100
    except Exception as e:
        print(f"Error fetching monthly data for {symbol}: {e}")
    return None

def get_annual_change(symbol):
    start_of_year = datetime(datetime.now().year - 1, 1, 1)
    try:
        annual_data = si.get_data(symbol, start_date=start_of_year.date())
        if not annual_data.empty:
            year_open = annual_data['open'][0]
            year_close = annual_data['close'][-1]
            return ((year_close - year_open) / year_open) * 100
    except Exception as e:
        print(f"Error fetching annual data for {symbol}: {e}")
    return None

def get_daily_and_ytd_percentage_changes():
    symbols = ['AAPL',	'MSFT',	'AMZN',	'NVDA',	'GOOGL',	'TSLA',	'GOOG',	'META',	'BRK-B',	'XOM',	'UNH',	'LLY',	'JPM',	'JNJ',	'V',	'PG',	'AVGO',	'MA',	'CVX',	'HD',	'ABBV',	'MRK',	'COST',	'PEP',	'WMT',	'ADBE',	'KO',	'CSCO',	'ACN',	'CRM',	'TMO',	'MCD',	'BAC',	'CMCSA',	'LIN',	'PFE',	'NFLX',	'ABT',	'ORCL',	'DHR',	'AMD',	'WFC',	'COP',	'DIS',	'INTC',	'AMGN',	'TXN',	'INTU',	'PM',	'CAT',	'VZ',	'IBM',	'HON',	'UNP',	'QCOM',	'NEE',	'LOW',	'BMY',	'GE',	'SPGI',	'AMAT',	'NOW',	'BA',	'UPS',	'BKNG',	'NKE',	'T',	'GS',	'RTX',	'ELV',	'DE',	'SBUX',	'MS',	'MDT',	'PLD',	'ISRG',	'TJX',	'ADP',	'MMC',	'MDLZ',	'GILD',	'LMT',	'BLK',	'VRTX',	'SYK',	'CVS',	'REGN',	'AXP',	'CB',	'ADI',	'SLB',	'ETN',	'CI',	'PGR',	'LRCX',	'SCHW',	'ZTS',	'C',	'BSX',	'BX',	'EOG',	'BDX',	'MU',	'AMT',	'MO',	'TMUS',	'CME',	'SO',	'PANW',	'DUK',	'FI',	'SNPS',	'LULU',	'AON',	'EQIX',	'ITW',	'APD',	'PYPL',	'CDNS',	'NOC',	'ICE',	'HUM',	'MPC',	'KLAC',	'FDX',	'CSX',	'MCK',	'SHW',	'CL',	'ABNB',	'WM',	'EMR',	'ORLY',	'PXD',	'PSX',	'FCX',	'MMM',	'ROP',	'VLO',	'NXPI',	'TGT',	'PH',	'USB',	'GD',	'CMG',	'HCA',	'AJG',	'MCO',	'APH',	'F',	'MAR',	'PNC',	'TDG',	'CARR',	'AZO',	'TT',	'MSI',	'ANET',	'NSC',	'GM',	'PCAR',	'CHTR',	'HES',	'SRE',	'OXY',	'AIG',	'ADSK',	'EW',	'ECL',	'PSA',	'MCHP',	'AFL',	'CTAS',	'WELL',	'WMB',	'KMB',	'ADM',	'MSCI',	'STZ',	'ON',	'MET',	'MNST',	'HLT',	'AEP',	'CCI',	'EXC',	'NUE',	'TRV',	'D',	'TEL',	'HAL',	'CNC',	'FTNT',	'OKE',	'GIS',	'CPRT',	'PAYX',	'BIIB',	'TFC',	'ROST',	'JCI',	'IQV',	'COF',	'BKR',	'IDXX',	'CTVA',	'DOW',	'ODFL',	'CEG',	'DXCM',	'SPG',	'DLR',	'O',	'KVUE',	'VRSK',	'CTSH',	'PCG',	'PRU',	'AME',	'YUM',	'DD',	'AMP',	'LHX',	'FIS',	'SYY',	'MRNA',	'BK',	'A',	'OTIS',	'ROK',	'DHI',	'CMI',	'EL',	'KMI',	'KDP',	'FAST',	'XEL',	'DVN',	'GWW',	'CSGP',	'COR',	'URI',	'HSY',	'ACGL',	'PPG',	'GPN',	'ED',	'NEM',	'RSG',	'ALL',	'EA',	'VICI',	'KR',	'PEG',	'LEN',	'FANG',	'WST',	'PWR',	'IT',	'APTV',	'VMC',	'KHC',	'GEHC',	'CDW',	'FTV',	'IR',	'ANSS',	'EXR',	'WEC',	'MLM',	'EIX',	'AWK',	'WBD',	'LYB',	'MTD',	'AVB',	'DAL',	'TROW',	'KEYS',	'ZBH',	'DG',	'GLW',	'EBAY',	'CBRE',	'WY',	'CHD',	'CAH',	'HPQ',	'EFX',	'TSCO',	'WTW',	'HPE',	'FICO',	'DLTR',	'HIG',	'RMD',	'TTWO',	'RCL',	'XYL',	'ALGN',	'STE',	'BR',	'DFS',	'STT',	'SBAC',	'MPWR',	'ILMN',	'DTE',	'MTB',	'CTRA',	'ES',	'GPC',	'EQR',	'ETR',	'DOV',	'AEE',	'ULTA',	'TDY',	'NVR',	'TRGP',	'MOH',	'WAB',	'FLT',	'ALB',	'BAX',	'RJF',	'MKC',	'INVH',	'FE',	'LH',	'HWM',	'VRSN',	'PPL',	'IRM',	'J',	'IFF',	'CNP',	'DRI',	'HOLX',	'FSLR',	'EXPD',	'BRO',	'FDS',	'FITB',	'VTR',	'MRO',	'STLD',	'BG',	'PTC',	'EG',	'CINF',	'ENPH',	'NDAQ',	'AKAM',	'CBOE',	'CF',	'WAT',	'PHM',	'TYL',	'PFG',	'CLX',	'LUV',	'RF',	'GRMN',	'ATO',	'NTAP',	'TXT',	'COO',	'K',	'IEX',	'CMS',	'SWKS',	'ARE',	'JBHT',	'LVS',	'BALL',	'WBA',	'TER',	'MAA',	'EPAM',	'AVY',	'OMC',	'HBAN',	'EQT',	'TSN',	'WDC',	'NTRS',	'CCL',	'EXPE',	'UAL',	'DGX',	'AXON',	'PKG',	'RVTY',	'SNA',	'POOL',	'ESS',	'DPZ',	'AMCR',	'BBY',	'APA',	'LW',	'WRB',	'CAG',	'LKQ',	'SJM',	'SWK',	'SYF',	'KMX',	'LDOS',	'STX',	'PAYC',	'CE',	'TRMB',	'LNT',	'CFG',	'IP',	'MAS',	'NDSN',	'L',	'EVRG',	'MOS',	'TAP',	'ZBRA',	'VTRS',	'LYV',	'HST',	'PODD',	'MTCH',	'IPG',	'HRL',	'INCY',	'UDR',	'JKHY',	'KIM',	'AES',	'TECH',	'PNR',	'ROL',	'MGM',	'CDAY',	'BF.B',	'NI',	'GEN',	'CHRW',	'CPT',	'CRL',	'PEAK',	'CZR',	'REG',	'KEY',	'HSIC',	'GL',	'BWA',	'FFIV',	'QRVO',	'TFX',	'ALLE',	'WRK',	'EMN',	'WYNN',	'NRG',	'JNPR',	'PNW',	'HAS',	'CTLT',	'AAL',	'FMC',	'CPB',	'AOS',	'BXP',	'HII',	'FOXA',	'RHI',	'AIZ',	'UHS',	'ETSY',	'MKTX',	'NWSA',	'BIO',	'BBWI',	'XRAY',	'SEDG',	'WHR',	'BEN',	'GNRC',	'NCLH',	'FRT',	'TPR',	'IVZ',	'PARA',	'VFC',	'CMA',	'DVA',	'ZION',	'RL',	'SEE',	'ALK',	'MHK',	'OGN',	'DXC',	'FOX',	'NWS']  # ... add all S&P 500 symbols here

    changes = []

    for symbol in symbols:
        try:
            data = si.get_data(symbol, start_date=datetime.now().date())
            if not data.empty:
                open_price = data['open'][0]
                close_price = data['close'][0]
                change = ((close_price - open_price) / open_price) * 100
                changes.append((symbol, change))
        except Exception as e:
            print(f"Error fetching data for {symbol}: {e}")

    changes.sort(key=lambda x: x[1], reverse=True)

    top_10_gains = changes[:10]
    top_10_losses = changes[-10:]
    top_10_losses.sort(key=lambda x: x[1])

    current_year = datetime.now().year
    start_date = f"{current_year}-01-01"

    def get_ytd_change(symbol):
        try:
            ytd_data = si.get_data(symbol, start_date=start_date)
            if not ytd_data.empty:
                ytd_open = ytd_data['open'][0]
                ytd_close = ytd_data['close'][-1]
                return ((ytd_close - ytd_open) / ytd_open) * 100
        except Exception as e:
            print(f"Error fetching YTD data for {symbol}: {e}")
        return None
    for i in range(len(top_10_gains)):
        symbol = top_10_gains[i][0]
        ytd_change = get_ytd_change(symbol)
        monthly_change = get_monthly_change(symbol)
        annual_change = get_annual_change(symbol)
        beta, market_cap, dividend_yield = get_stock_info(symbol)
        top_10_gains[i] += (ytd_change, monthly_change, annual_change, beta, market_cap, dividend_yield)

    for i in range(len(top_10_losses)):
        symbol = top_10_losses[i][0]
        ytd_change = get_ytd_change(symbol)
        monthly_change = get_monthly_change(symbol)
        annual_change = get_annual_change(symbol)
        beta, market_cap, dividend_yield = get_stock_info(symbol)
        top_10_losses[i] += (ytd_change, monthly_change, annual_change, beta, market_cap, dividend_yield)

    return top_10_gains, top_10_losses

@app.route('/')
def index():
    top_10_gains, top_10_losses = get_daily_and_ytd_percentage_changes()

    # Prepare the data for the graphs
    gains_data = {
        'symbols': [x[0] for x in top_10_gains],
        'daily': [x[1] for x in top_10_gains],
        'ytd': [x[2] for x in top_10_gains],
        'monthly': [x[3] for x in top_10_gains],
        'annual': [x[4] for x in top_10_gains]
    }

    losses_data = {
        'symbols': [x[0] for x in top_10_losses],
        'daily': [x[1] for x in top_10_losses],
        'ytd': [x[2] for x in top_10_losses],
        'monthly': [x[3] for x in top_10_losses],
        'annual': [x[4] for x in top_10_losses]
    }

    # Create combined graphs for top 10 gains
    gains_fig = go.Figure()
    for key in ['daily', 'ytd', 'monthly', 'annual']:
        gains_fig.add_trace(go.Bar(name=key.capitalize(), x=gains_data['symbols'], y=gains_data[key]))
    gains_fig.update_layout(barmode='group', title="Top 10 Gains - Daily, YTD, Monthly, Annual")

    # Create combined graphs for top 10 losses
    losses_fig = go.Figure()
    for key in ['daily', 'ytd', 'monthly', 'annual']:
        losses_fig.add_trace(go.Bar(name=key.capitalize(), x=losses_data['symbols'], y=losses_data[key]))
    losses_fig.update_layout(barmode='group', title="Top 10 Losses - Daily, YTD, Monthly, Annual")

    # Generate div elements for the graphs
    gains_div = pyo.plot(gains_fig, output_type='div', include_plotlyjs=False)
    losses_div = pyo.plot(losses_fig, output_type='div', include_plotlyjs=False)

    # Return the HTML template with the graph divs
    template = """
    <html>
    <head>
        <title>Stock Changes</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    </head>
    <body>
        <div class="container mt-5">
            <h1>Top 10 Gains - Daily, YTD, Monthly, Annual:</h1>
            {{ gains_div|safe }}
            <div>
                {% for stock in top_10_gains %}
                <p>{{ stock[0] }} - Beta: {{ stock[5] }}, Market Cap: {{ stock[6] }}, Dividend Yield: {{ stock[7] }}</p>
                {% endfor %}
            </div>

            <h1>Top 10 Losses - Daily, YTD, Monthly, Annual:</h1>
            {{ losses_div|safe }}
            <div>
                {% for stock in top_10_losses %}
                <p>{{ stock[0] }} - Beta: {{ stock[5] }}, Market Cap: {{ stock[6] }}, Dividend Yield: {{ stock[7] }}</p>
                {% endfor %}
            </div>
        </div>
    </body>
    </html>
    """
    
    return render_template_string(template, gains_div=gains_div, losses_div=losses_div, top_10_gains=top_10_gains, top_10_losses=top_10_losses)

if __name__ == '__main__':
    app.run(debug=True)
