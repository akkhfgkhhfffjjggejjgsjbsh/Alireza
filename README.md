using System;
using cAlgo.API;
using cAlgo.API.Indicators;
using cAlgo.API.Internals;

namespace cAlgo.Robots
{
    [Robot(TimeZone = TimeZones.UTC, AccessRights = AccessRights.None)]
    public class NewYorkCloseTradingBot : Robot
    {
        private DateTime _newYorkCloseTime;
        private double _newYorkClosePrice;

        [Parameter("Lot Size", DefaultValue = 0.1)]
        public double LotSize { get; set; }

        [Parameter("Start Trading Hour (Server Time)", DefaultValue = 17)]
        public int StartHour { get; set; }

        protected override void OnStart()
        {
            // Set the New York close time (assuming it is 21:00 UTC)
            _newYorkCloseTime = new DateTime(DateTime.UtcNow.Year, DateTime.UtcNow.Month, DateTime.UtcNow.Day, 21, 0, 0);
        }

        protected override void OnBar()
        {
            // Check if it is the start of a new day
            if (Server.Time.Hour == 0 && Server.Time.Minute == 0)
            {
                // Get the closing price of the previous day
                _newYorkClosePrice = MarketSeries.Close.Last(1);
            }

            // Check if current server time is after the start hour
            if (Server.Time.Hour >= StartHour)
            {
                ExecuteTrade();
            }
        }

        private void ExecuteTrade()
        {
            if (Positions.Count > 0)
                return; // Ensure no open positions

            // Check the current price against New York close price
            double currentPrice = Symbol.Bid;
            if (currentPrice > _newYorkClosePrice)
            {
                // Open a sell position
                ExecuteMarketOrder(TradeType.Sell, SymbolName, Symbol.QuantityToVolumeInUnits(LotSize), "Sell after NY close", null, null, 0, "Sell Position");
            }
            else if (currentPrice < _newYorkClosePrice)
            {
                // Open a buy position
                ExecuteMarketOrder(TradeType.Buy, SymbolName, Symbol.QuantityToVolumeInUnits(LotSize), "Buy after NY close", null, null, 0, "Buy Position");
            }
        }
    }
}
