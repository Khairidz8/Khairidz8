//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
input double ATRMultiplier = 2.0;        // Multiplier for ATR-based lot size
input int ATRPeriod = 14;                // ATR period
input double RiskPercent = 1.0;          // Risk percentage per trade
input double TakeProfitATRMultiplier = 3.0; // ATR multiplier for Take Profit
input int HalfTrendPeriod = 2;           // Period for Half Trend
input int MagicNumber = 12345;           // EA magic number
double LotSize;

//+------------------------------------------------------------------+
//| Calculate lot size based on ATR and account balance               |
//+------------------------------------------------------------------+
double CalculateLotSize() {
   double atr = iATR(NULL, 0, ATRPeriod, 0);   // Get ATR value
   double riskAmount = AccountBalance() * (RiskPercent / 100); // Risk in money
   double stopLossPips = atr * ATRMultiplier; // Stop loss in pips
   double lotSize = riskAmount / stopLossPips / MarketInfo(Symbol(), MODE_TICKVALUE);
   
   return NormalizeDouble(lotSize, 2); // Normalize to 2 decimal places
}

//+------------------------------------------------------------------+
//| HalfTrend Indicator function (custom calculation or include)      |
//+------------------------------------------------------------------+
int HalfTrendSignal() {
   // Implement or use custom Half Trend indicator logic here
   // Return 1 for Buy, -1 for Sell, 0 for no signal
   // Example of custom logic (or use an existing HalfTrend indicator)
   
   double halfTrendUp = iCustom(NULL, 0, "HalfTrend", HalfTrendPeriod, 1, 0); // Example custom indicator
   double halfTrendDown = iCustom(NULL, 0, "HalfTrend", HalfTrendPeriod, 2, 0);
   
   if (halfTrendUp > 0 && halfTrendDown == 0) return 1; // Buy signal
   if (halfTrendDown > 0 && halfTrendUp == 0) return -1; // Sell signal
   
   return 0; // No signal
}

//+------------------------------------------------------------------+
//| Take Profit calculation based on ATR                              |
//+------------------------------------------------------------------+
double CalculateTakeProfit(int direction) {
   double atr = iATR(NULL, 0, ATRPeriod, 0);   // Get ATR value
   if (direction == 1) {
      // Take profit for Buy order
      return Ask + (atr * TakeProfitATRMultiplier);
   } else if (direction == -1) {
      // Take profit for Sell order
      return Bid - (atr * TakeProfitATRMultiplier);
   }
   return 0;
}

//+------------------------------------------------------------------+
//| Expert tick function                                              |
//+------------------------------------------------------------------+
void OnTick() {
   // Check for existing orders
   if (OrdersTotal() > 0) return;

   // Get HalfTrend signal
   int signal = HalfTrendSignal();
   
   if (signal != 0) {
      // Calculate lot size
      LotSize = CalculateLotSize();

      // Calculate take profit level
      double takeProfit = CalculateTakeProfit(signal);
      
      // Execute buy or sell based on the signal
      if (signal == 1) {
         // Buy order
         OrderSend(Symbol(), OP_BUY, LotSize, Ask, 3, 0, takeProfit, "HalfTrend Buy", MagicNumber, 0, Blue);
      } else if (signal == -1) {
         // Sell order
         OrderSend(Symbol(), OP_SELL, LotSize, Bid, 3, 0, takeProfit, "HalfTrend Sell", MagicNumber, 0, Red);
      }
   }
}
