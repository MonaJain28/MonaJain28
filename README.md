
from flask import Flask, request, jsonify

app = Flask(__name__)

# Initialize user balances
user_balances = {
    'u1': 0,
    'u2': 0,
    'u3': 0,
    'u4': 0,
}

@app.route('/add_expense', methods=['POST'])
def add_expense():
    data = request.json

    payer = data['payer']
    amount = data['amount']
    participants = data['participants']
    split_type = data['split_type']
    
    if split_type == 'EQUAL':
        # Calculate the share for each participant
        share = amount / len(participants)

        # Update balances
        user_balances[payer] += amount
        for participant in participants:
            if participant != payer:
                user_balances[participant] -= share

    elif split_type == 'EXACT':
        # Get exact amounts for each participant
        exact_amounts = data['exact_amounts']

        # Update balances
        user_balances[payer] += amount
        for participant, exact_amount in zip(participants, exact_amounts):
            if participant != payer:
                user_balances[participant] -= exact_amount

    elif split_type == 'PERCENT':
        # Get percentage shares for each participant
        percentages = data['percentages']

        # Calculate and update balances based on percentages
        for participant, percent_share in zip(participants, percentages):
            share = amount * (percent_share / 100)
            user_balances[participant] -= share
            user_balances[payer] += share

    else:
        return jsonify({'error': 'Invalid split type'}), 400

    return jsonify({'message': 'Expense added successfully', 'balances': user_balances})

if __name__ == '__main__':
    app.run(debug=True)

#json for example 1 : Split Equally
{
    "payer": "u1",
    "amount": 1000,
    "participants": ["u1", "u2", "u3", "u4"],
    "split_type": "EQUAL"
}

#json for example 2. Exact Amounts
{
    "payer": "u1",
    "amount": 1250,
    "participants": ["u1", "u2", "u3"],
    "split_type": "EXACT",
    "exact_amounts": [0, 370, 880]
}

#json for example 3. Percentage Split
{
    "payer": "u4",
    "amount": 1200,
    "participants": ["u1", "u2", "u3", "u4"],
    "split_type": "PERCENT",
    "percentages": [40, 20, 20, 20]
}
