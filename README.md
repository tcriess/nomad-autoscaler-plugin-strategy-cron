# Nomad Autoscaler Cron Strategy

A cron-like strategy plugin, where task groups are scaled based on a predefined scheduled and counts can be arbitrary expressions on the evaluated values.

```hcl
job "webapp" {
  ...
  group "demo" {
    ...
    scaling {
      ...
      policy {
        check "business hours" {
          source = "prometheus"
          query = "..."

          strategy "cron" {
            count = 2
            hysteresis = "2,4,6"
            expression_a = "MetricsMax > 5 ? 7 : 5"
            period_business = "* * 9-17 * * mon-fri * -> a"
            period_weekend = "* * * * * sat,sun * -> 1"
          }
        }
      }
    }
  }
}
```

In the example above, every weekday between 9 and 17:59, the number of instances is increased to 5 or 7 (depending on the max of the evaluated time series) to handle the large traffic during operating hours. 
On the weekend, the number drops to 1.
The rest of the time, the value is taken from the default `count = 2`.
Additionally, a hysteresis can be defined via a comma-separated list of integers or expressions.
If the current count is equal or above one of the hysteresis values, the target count cannot drop between the preceding hysteresis value and the current hysteresis value.
