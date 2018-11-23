# ONZO Software Test Engineering Challenge

In this challenge we will focus on our core data product: ONZO's [Engagement API](http://apidocs.onzodata.com/engagement/v2/). 

## Context

We are developing a new feature that will add more information to the "Breakdown Details" response.

The main user story (requirement) has been written like this:

```
As an Onzo API User,
I would like to know more about what's going on in each customer's house,
So I can help them plan for their energy expenditure over the winter months.
```

The product owner has a list of items we need to add, and the first priority item is "hair dryers".

## Task 1
As a test engineer, please:

1. Prepare a test plan for this new feature (use whiteboard / notebook / paper as you see fit). How will you interact with 
the product owner, software engineering and customer success team?

2. Considering the following function, prepare a set of automated test cases to cover what you consider critical.

```python

 @staticmethod
    def _get_breakdown_data(request_args, onzo_household_id, organisation_id, valid_from,
                            aggregation):
        if request_args['fuel'] == 'elec':
            data_repo = electric_breakdown_repository()
        elif request_args['fuel'] == 'gas':
            data_repo = gas_breakdown_repository()
        else:
            return None

        if 'date_from' in request_args:
            date_from = request_args['date_from']
        elif aggregation.endswith('day'):
            past_days = int(re.match('^([1-9][0-9]*)day$', aggregation).group(1)) - 1
            date_from = request_args['date_to'] - timedelta(days=past_days)
        elif aggregation.endswith('month'):  # get number of days to subtract to get 1st of the month
            past_days = request_args['date_to'].day - 1
            date_from = request_args['date_to'] - timedelta(days=past_days)
        else:
            return None

        result = data_repo.get_breakdown_range(
            organisation_id, onzo_household_id,
            date_from=date_from.strftime('%Y-%m-%d'),
            date_to=(request_args['date_to'] + timedelta(days=1)).strftime('%Y-%m-%d')
        )
        valid_from_str = valid_from.strftime('%Y-%m-%d')
        result = [day.categories for day in result if str(day.day) >= valid_from_str]
        return combine_breakdown_list(result)
)
```

## Task 2
1. Here is some sample API output from a request 
to: `https://api.onzo.io/engagement/v2/breakdown/AGJLLCFHEJE/month/2018-11-09?fuel=elec&units=energy`  
(_before the new feature has been implemented_):

```json

{
    "date": "2018-11-09",
    "message": "",
    "range": "Month",
    "title": "Month To 2018-11-09",
    "titles": {},
    "values": {
        "always_on": [
            11.286828278545796,
            14908.320000000002
        ],
        "cooking": [
            8.696893195499168,
            11487.378346207031
        ],
        "cooling": [
            0,
            0
        ],
        "heating": [
            30.097059140814636,
            39754.00153673642
        ],
        "home_entertainment": [
            6.453551883362926,
            8524.238540658755
        ],
        "laundry_dishwashing": [
            12.07222060594808,
            15945.713309572582
        ],
        "lighting": [
            4.015136157784962,
            5303.432745371844
        ],
        "other": [
            2.5103555973721536,
            3315.828294344983
        ],
        "refrigeration": [
            4.521054464515543,
            5971.68
        ],
        "water_heating": [
            20.346900676156736,
            26875.407227108386
        ]
    }
}
```


a) How would you expect the data to look _after_ the feature has been implemented?
b) How will you ensure the customer is getting what they want?

2. Write an automated test to ensure the response is including the new feature.
3. Bonus: what edge cases could be present? How would you defend for them with code?

## Ground Rules

* Use the API documentation linked above.
* Feel free to use any online resources you need.
* Communicate your thought process and engage with the interviewer as needed.
* This is not a test - please think of it as a normal working day! Take your time.