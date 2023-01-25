# Pace Passport Index

The most well-known ranking system for the desirability of passports is the [Henley Passport Index](https://www.henleyglobal.com/passport-index), which counts the number of countries that a given passport allows entry into without a visa. The problem with the Henley index is that it implicitly gives all countries equal weighting, while realistically it is worth more to be able to travel to a country like the USA than to a country like Togo.

Another index is the [Nomad Capitalist Passport Index](https://nomadcapitalist.com/wp-content/uploads/2022/03/Nomad-Passport-Index-2022.pdf), where ratings are a mix of visa-free travel (50%), taxation of citizens (20%), perception of the country's citizens (10%, e.g. the World Happiness Report), the country's openness to dual citizenship (10%), and personal freedom (10%).

A major concern of the Pace index is not to rate countries themselves (e.g. citizens’ quality of life), but only their value as a travel destination. With this in mind, we have opted for parsimony in the variables used — namely, GDP and level of tourism. However, we also intend to release a web app where users will be able to choose from a menu of variables, and weight them according to what aspects of a country they value most.

A further aspect setting the Pace index apart is that it is fully transparent and replicable. Our data has been downloaded without modification, and the accompanying R program cleans the data and reconstructs the index from scratch each time it is run.

## Variables

### Destination Countries

For a list of visa-free destinations for a given passport, we use the dataset [here](https://github.com/ilyankou/passport-index-dataset), based on [passportindex.org](https://www.passportindex.org). We consider a country to not allow visa-free entry if its status is 'visa required', 'e-visa', 'no admission', or 'covid ban' (which exclusively affects Azerbaijan and China). It should be noted that this list of destinations is not exactly the same as that used by the Henley index.

Note that we include each country as a destination to itself, which seems reasonable when considering its value as a travel location.

### GDP 

Naturally, the size of a country's economy affects its desirability as a travel destination. This data has been downloaded from the [World Bank](https://data.worldbank.org/indicator/NY.GDP.MKTP.KD), and is measured in constant 2015 US dollars, which will make it easier to compare the index in future years to that of the present year. 

The most recent year is 2021, and for countries missing data we use the latest year available, back to 2015. Since we are mainly interested in order of magnitude, and since most countries consistently missing GDP data are small, we feel this is an acceptable approximation.

To ensure that a country's GDP score is bounded, we divide it by the highest GDP in our sample, namely the USA. Hence the USA gets a full score of 1, while all other countries are graded relative to that, with a theoretical lower bound of zero.

### Tourism

We use data from the [World Bank](https://data.worldbank.org/indicator/ST.INT.ARVL) on tourism by number of arrivals. However, the COVID pandemic sharply reduced tourism for most countries, so we use tourism data from 2019, or the most recent year available, back to 2013.

Like with GDP, we bound each country's tourism score between 0 and 1 by dividing it by the highest level in our sample, namely of France. 

For countries missing data up to our cut-off points, we drop them from the sample. This is another reason why our list of destination countries differs from Henley's list. Most of these missing countries (e.g. North Korea) would be near the bottom of the list anyways.

### Henley Ranking

These were copy-pasted directly from the Henley [website](https://www.henleyglobal.com/passport-index/ranking) for the year 2022, and cleaned up into the attached CSV file. These do not contribute to a country's score, but it is informative to compare their unweighted rankings to our own.

### Weights

At the very beginning of the program, the user manually sets the variable <code>w.gdp</code>, which shows what proportion of the <code>total</code> (set to 10 for convenience) comes from either GDP or tourism. We found that setting the weight too high (<code>w.gdp=0.75</code>, so that 7.5 points out of 10 in <code>score</code> come from GDP) caused several dark horses to drastically rise in the ranks, even though they didn't seem to belong there. Conversely, setting the weight low (<code>w.gdp=0.25</code>) led to only negligible changes. Hence we opt for an even weighting of <code>0.5</code> as a default.

-----

Running the source file will create a large list <code>destinations</code> of the destination countries for each passport. 

It also creates the data frame <code>data</code>, which contains the major variables of interest: 
- <code>country</code> is a given country's passport
- <code>henley</code> is the original Henley score 
- <code>h.rank</code> is the Henley ranking (which includes many ties)
- <code>gdp</code> is GDP<sub><small>i</small></sub> / GDP<sub><small>USA</small></sub>
- <code>tourism</code> is tourism<sub><small>i</small></sub> / tourism<sub><small>France</small></sub> 
- <code>score</code> is that country's individual score, which (for a specified weight *w*) is equal to *w*×<code>gdp</code> + (1 - *w*)×<code>tourism</code>
- <code>n.dest</code> is the number of destination countries for a given passport (which differs from <code>henley</code> due to data issues)
- <code>index</code> is the result of adding up the <code>score</code>s for each destination country

## Future Directions

This first version of our index is highly experimental, and we intend to tinker with it in future iterations.

The current version assigns each country a specific score. An interesting way to extend this would be bilateral variables, where each destination country is weighted relative to the origin country. For example, a geographically closer destination should get a higher score than one further away. (There are some tricky methodological issues here: where do we start and stop our distance measurement?) Further, these need not be symmetric — a US citizen may not want to travel to Russia, even though a Russian citizen may be fine traveling to the US, for instance.

To the obvious criticism that the current index doesn't have enough variables, our web app will offer a larger menu to choose from. Playing around with this will afford much better intuitions about what affects the rankings, which will no doubt make their way into our next version.
