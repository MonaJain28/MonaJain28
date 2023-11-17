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

    # Calculate the share for each participant
    share = amount / len(participants)

    # Update balances
    user_balances[payer] += amount
    for participant in participants:
        if participant != payer:
            user_balances[participant] -= share

    return jsonify({'message': 'Expense added successfully', 'balances': user_balances})

if __name__ == '__main__':
    app.run(debug=True)

