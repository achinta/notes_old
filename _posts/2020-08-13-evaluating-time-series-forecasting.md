## Purpose
What are the options to do time series forecasting at scale (say millions of series), where we output yhat, yhat_upper and yhat_lower?

 * Facebook Prophet
 * [Neural Prophet](https://neuralprophet.com)
 * [Multivariate Probabilistic Time Series Forecasting via Conditioned Normalizing Flows](https://arxiv.org/abs/2002.06103)
 * [Pytorch Forecasting](https://pytorch-forecasting.readthedocs.io/en/latest/)
 * [flow-forecast](https://github.com/AIStream-Peelout/flow-forecast)

### Facebook Prophet
##### [Where Facebook Shines](https://research.fb.com/blog/2017/02/prophet-forecasting-at-scale/)
 - hourly, daily, or weekly observations with at least a few months (preferably a year) of history
 - strong multiple “human-scale” seasonalities: day of week and time of year
 - important holidays that occur at irregular intervals that are known in advance (e.g. the Super Bowl)
 - a reasonable number of missing observations or large outliers
 - historical trend changes, for instance due to product launches or logging changes
 - trends that are non-linear growth curves, where a trend hits a natural limit or saturates
#### How Prophet Works
 - A piecewise linear or logistic growth curve trend. Prophet automatically detects changes in trends by selecting changepoints from the data.
 - A yearly seasonal component modeled using Fourier series.
 - A weekly seasonal component using dummy variables.
 - A user-provided list of important holidays.

> The important idea in Prophet is that by doing a better job of fitting the trend component very flexibly, we more accurately model seasonality and the result is a more accurate forecast. 

> By default, Prophet will provide uncertainty intervals for the trend component by simulating future trend changes to your time series. If you wish to model uncertainty about future seasonality or holiday effects, you can run a few hundred HMC iterations (which takes a few minutes) and your forecasts will include seasonal uncertainty estimates.

> We fit the Prophet model using Stan, and have implemented the core of the Prophet procedure in Stan’s probabilistic programming language. Stan performs the MAP optimization for parameters extremely quickly (<1 second), gives us the option to estimate parameter uncertainty using the Hamiltonian Monte Carlo algorithm, 
