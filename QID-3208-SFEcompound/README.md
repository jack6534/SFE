
[<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/banner.png" width="888" alt="Visit QuantNet">](http://quantlet.de/)

## [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/qloqo.png" alt="Visit QuantNet">](http://quantlet.de/) **SFEcompound** [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/QN2.png" width="60" alt="Visit QuantNet 2.0">](http://quantlet.de/)

```yaml

Name of QuantLet : SFEcompound

Published in : Statistics of Financial Markets

Description : 'Computes European Call-on-a-Call option prices (compound option, Option on Option)
using a binomial tree for assets without dividends.'

Keywords : 'asset, binomial, binomial-tree, black-scholes, call, discrete, european-option,
exotic-option, financial, option, option-price, price, simulation, stock-price'

See also : SFEasian, SFElookback

Author : Awdesch Melzer

Submitted : Wed, June 10 2015 by Lukas Borke

```


### R Code:
```r

rm(list = ls(all = TRUE))
graphics.off()

# parameter settings
K1    = 20            # Exercise price compound
K2    = 210           # Exercise price option
St1   = 230           # Price of underlying asset
r     = 0.04545       # dom. interest rate
sigma = 0.25          # volatility per year
T2    = 1             # time to maturity option
T1    = 0.5           # time to maturity compound
b     = r             # cost of carry out
dt    = 0.2           # Interval of step
n     = floor(T2/dt)  # number of steps

# parameters from equation (7.2)
u  = exp(sigma * sqrt(dt))                              # upward proportion: approx 1.1183  
d  = 1/u                                                # downward proportion approx. 0.89422
p  = 0.5 + 0.5 * (b - 0.5 * sigma^2) * sqrt(dt)/sigma   # Pseudo probability of up movement approx 0.5127  
un = matrix(0, n + 1, 1)
un[n + 1, 1] = 1
dm           = t(un)    # down movement
um           = dm       # up movement

j = 1
while (j < n + 1) {
    # Down movement dynamics
    d1 = c(matrix(0, 1, n - j), (matrix(1, 1, j + 1) * d)^(seq(0:j) - 1))
    dm = rbind(dm, d1)
    # Up movement dynamics
    u1 = c(matrix(0, 1, n - j), matrix(1, 1, j + 1) * u^seq(j, 0, -1))
    um = rbind(um, u1)
    j = j + 1
}

um = t(um)
dm = t(dm)

# Stock price development
s = St1 * um * dm
colnames(s) = c()

print("Stock price development")
print(s)

# Rearrangement
s = s[ncol(s):1, ]
opt = matrix(0, ncol(s), ncol(s))
# Determine option values from prices
opt[, n + 1] = apply(cbind(as.matrix(s[, ncol(s)] - K2), matrix(0, nrow(as.matrix(s[, ncol(s)] - K2)), 1)), 1, max)

for (j in n:1) {
    l = 1:j
    # Probable option values discounted back one time step
    discopt = ((1 - p) * opt[l, j + 1] + p * opt[l + 1, j + 1]) * exp(-b * dt)
    # Option value is max of current stock price - strike or discopt
    opt[, j] = c(discopt, matrix(0, n + 1 - j, 1))
}

European_Call_Price = opt[ncol(opt):1, ]

print("Option price development")
print(European_Call_Price)

print("The price of the European call option at time t_0 is")
print(European_Call_Price[n + 1, 1])

# Second step: Call on Call
C	= opt[1:(floor(T1/dt) + 1), 1:(floor(T1/dt) + 1)]
CC	= opt[1:(floor(T1/dt) + 2), 1:(floor(T1/dt) + 2)]
CC[, (floor(T1/dt) + 2)] = 0

CO = matrix(0, ncol(C), ncol(C))
CO[, (floor(T1/dt) + 1)] = apply(cbind(as.matrix(C[, (floor(T1/dt) + 1)] - 
    K1), 0), 1, max) + apply(cbind(as.matrix(CC[(floor(T1/dt) + 2), (floor(T1/dt) + 2)] - K1), 0), 1, max)

k = floor(T1/dt)

# employing equation (7.5)
for (i in k:1) {
    j = i:1
    CO[j, i] = exp(-r * dt) * (p * CO[j + 1, i + 1] + (1 - p) * CO[j, i + 1])
}

European_Call_on_Call_Price = CO[ncol(CO):1, ]

# final output
print("Call on Call Option price development")
print(European_Call_on_Call_Price)

print("The price of the European call on call (compound) option at time t_0 is")
print(European_Call_on_Call_Price[(floor(T1/dt) + 1), 1]) 

```
