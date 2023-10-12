# NoSQL Database Design 

In this lab, our client, a gaming company, has requested improvements for their central cafeteria dashboard. They aim to display real-time updates for the top five players in each game. However, they are facing issues with long loading times for initial player data on the dashboard. The company is already using DynamoDB, but they believe that implementing a secondary index could significantly enhance their data retrieval speed.

# Lab Architecture:

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/100aecf8-4bb3-433e-8a10-4dc64dd82c0b)

# Step 1: Setup and Code Review

- In the Cloud9 environment, I clondc the necessary Git repository and installed the required Python modules.
- I navigated to the dynamoDB-tests directory and executed sudo pip3 install -r requirements.txt to ensure the necessary Python packages were installed.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/69b6d5c2-10c1-4859-a240-fff78abfabb4)

# Step 2: Querying Data with player_score_get_item.py

- I opened the player_score_get_item.py file in the Cloud9 IDE, reviewed the code, and ran the script.
- The script queries DynamoDB for player data and displays the results in a tabular format.
- I noted the duration and consumed capacity of the query.

```python
import boto3
from tabulate import tabulate
import time

TABLE_NAME = 'player_score'
SECONDS_TO_MILIS = 1000


def print_player_data(get_item_result):
    player_data = get_item_result['Item']
    print(tabulate([convert_player_data_to_array(player_data)],
                   headers=['Name', 'Game', 'Score', 'Timestamp'],
                   tablefmt='pretty'))
    print('Total Items Returned: 1')


def convert_player_data_to_array(player_data):
    return [player_data['player_name'], player_data['game'],
            player_data['score'], player_data['game_timestamp']]


def print_capacity(get_item_result):
    print('Consumed Capacity: '
          + str(get_item_result['ConsumedCapacity']['CapacityUnits']))


def print_data(dynamo_result, total_duration):
    print_player_data(dynamo_result)
    print_capacity(dynamo_result)
    print('Duration: ' + str(total_duration) + ' miliseconds')


if __name__ == "__main__":
    dynamodb = boto3.resource('dynamodb')
    score_table = dynamodb.Table(TABLE_NAME)

    start_time = time.time()

    player_game_data = score_table.get_item(
        Key={
            'player_name': 'James Smith',
            'game_timestamp': 1461351017
        },
        ReturnConsumedCapacity='TOTAL')

    end_time = time.time()
    duration = int((end_time - start_time) * SECONDS_TO_MILIS)
    print_data(player_game_data, duration)
```

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/2b2f6739-74f4-4872-98d9-19dc89106015)

# Step 3: Querying Data with player_score_query.py

- Similar to Step 2, I opened the player_score_query.py file and uncommented lines 52-53 as per the lab instructions.
- This script queries player data, applies filtering, and displays the results in a tabular format.
- I recorded the duration and consumed capacity of the query.

```python
import boto3
from boto3.dynamodb.conditions import Attr, Key
from tabulate import tabulate
import time

TABLE_NAME = 'player_score'
SECONDS_TO_MILIS = 1000
MAX_RECORDS_TO_PRINT = 5

def print_player_data(dynamo_result, total_duration=None):
    items_to_print = dynamo_result['Items']
    total_records = len(items_to_print)
    formated_records = convert_player_data_to_array(items_to_print)
    print(tabulate(formated_records,
                   headers=['Name', 'Game', 'Score', 'Timestamp', 'Badges'],
                   tablefmt='pretty'))
    print('Printed {} of {} records returned'.format(len(formated_records), total_records))
    print_capacity(dynamo_result)
    print('Duration: ' + str(total_duration) + ' miliseconds')

def print_capacity(get_item_result):
    print('Consumed Capacity: '
          + str(get_item_result['ConsumedCapacity']['CapacityUnits']))


def convert_player_data_to_array(items_to_print):
    current_position = 0
    array_to_print = []
    while current_position < len(items_to_print) \
        and current_position < MAX_RECORDS_TO_PRINT:
        item_array = [items_to_print[current_position]['player_name'],
                      items_to_print[current_position]['game'],
                      items_to_print[current_position]['score'],
                      items_to_print[current_position]['game_timestamp']]
        if 'badge' in items_to_print[current_position]:
            item_array.append(items_to_print[current_position]['badge'])
        else:
            item_array.append("")
        array_to_print.append(item_array)
        current_position = current_position + 1
    return array_to_print


if __name__ == "__main__":
    dynamodb = boto3.resource('dynamodb')
    score_table = dynamodb.Table(TABLE_NAME)

    start_time = time.time()
    player_game_data = score_table.query(
        KeyConditionExpression=Key('player_name').eq('James Smith'),
        # Follow lab step to uncomment the following statements.
        # FilterExpression=Attr('badge').eq('Champs'),
        # Limit=5,
        ReturnConsumedCapacity='TOTAL')

    end_time = time.time()

    duration = int((end_time - start_time) * SECONDS_TO_MILIS)
    print_player_data(player_game_data, duration)

```

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/49014e43-3b0b-4e8c-a1eb-d96b0671611b)

# Step 4: Scanning Data with player_score_scan.py

- I explored the code in the player_score_scan.py file.
- This script performs a scan operation on DynamoDB, fetching player data and displaying it.
- I executed the script, recorded the duration, consumed capacity, and the printed records.

```python
import boto3
from boto3.dynamodb.conditions import Attr, Key
from tabulate import tabulate
import time

TABLE_NAME = 'player_score'
SECONDS_TO_MILIS = 1000
MAX_RECORDS_TO_PRINT = 10


def print_player_data(all_items, total_duration, total_consumed_capacity):
    total_records = len(all_items)
    formated_records = convert_player_data_to_array(all_items)
    print(tabulate(formated_records,
                   headers=['Name', 'Game', 'Score', 'Timestamp', 'Badges'],
                   tablefmt='pretty'))
    print('Printed {} of {} records returned'.format(len(formated_records),
                                                     total_records))
    print('Consumed Capacity: {}'.format(total_consumed_capacity))
    print('Duration: {} miliseconds'.format(total_duration))


def convert_player_data_to_array(items_to_print):
    current_position = 0
    array_to_print = []
    while current_position < len(items_to_print) \
        and current_position < MAX_RECORDS_TO_PRINT:
        item_array = [items_to_print[current_position]['player_name'],
                      items_to_print[current_position]['game'],
                      items_to_print[current_position]['score'],
                      items_to_print[current_position]['game_timestamp']]
        if 'badge' in items_to_print[current_position]:
            item_array.append(items_to_print[current_position]['badge'])
        else:
            item_array.append("")
        array_to_print.append(item_array)
        current_position = current_position + 1
    return array_to_print


if __name__ == "__main__":
    dynamodb = boto3.resource('dynamodb')
    score_table = dynamodb.Table(TABLE_NAME)

    start_time = time.time()
    total_consumed_capacity = 0
    all_items = list()

    player_game_data = score_table.scan(
        FilterExpression=Attr('badge').eq('Champs'),
        ReturnConsumedCapacity='TOTAL')
    total_consumed_capacity = player_game_data['ConsumedCapacity'] \
        ['CapacityUnits']
    all_items.extend(player_game_data['Items'])


    # If the result of the first batch is not complete. Pagination is required
    while 'LastEvaluatedKey' in player_game_data:
        player_game_data = score_table.scan(
            FilterExpression=Attr('badge').eq('Champs'),
            ExclusiveStartKey=player_game_data['LastEvaluatedKey'],
            ReturnConsumedCapacity='TOTAL')
        total_consumed_capacity = total_consumed_capacity + \
                                  player_game_data['ConsumedCapacity'][
                                      'CapacityUnits']
        all_items.extend(player_game_data['Items'])


    end_time = time.time()
    duration = int((end_time - start_time) * SECONDS_TO_MILIS)

    print_player_data(all_items, duration, total_consumed_capacity)
```

![5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a79f09b6-70d1-4697-8094-8b9ba499dee2)

# Step 5: Creating a Secondary Index

- I accessed the DynamoDB console to create a secondary index.
- The secondary index uses 'badge' as the partition key and 'game' as the sort key, both with string data types.
- This index should optimize queries based on badge and game attributes.

# Conclusion:

This lab focused on optimizing data retrieval from DynamoDB for a gaming company's cafeteria dashboard. We explored querying and scanning methods, emphasizing the importance of secondary indexes to improve query performance. By creating a secondary index based on specific attributes, we aim to address the long loading times experienced by the client's dashboard. This optimization will enhance the user experience and ensure real-time updates for the top players in each game.
