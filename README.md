```python
# Import necessary libraries from Flask
from flask import Flask, redirect, request, render_template, url_for

# Instantiate Flask application
app = Flask(__name__)

# Sample data representing transactions
transactions = [
    {'id': 1, 'date': '2023-06-01', 'amount': 100},
    {'id': 2, 'date': '2023-06-02', 'amount': -200},
    {'id': 3, 'date': '2023-06-03', 'amount': 300}
]

# Read operation: Route to list all transactions
@app.route("/")
def get_transactions():
    # Render the transactions list template and pass the transactions data
    return render_template("transactions.html", transactions=transactions)

# Create operation: Route to display and process add transaction form
@app.route("/add", methods=["GET", "POST"])
def add_transaction():
    if request.method == 'POST':
        # Extract form data to create a new transaction object
        transaction = {
            'id': len(transactions) + 1,         # Generate a new ID based on the current length of the transactions list
            'date': request.form['date'],        # Get the 'date' field value from the form
            'amount': float(request.form['amount']) # Get the 'amount' field value from the form and convert it to a float
        }

        # Append the new transaction to the transactions list
        transactions.append(transaction)

        # Redirect to the transactions list page after adding the new transaction
        return redirect(url_for("get_transactions"))

    # Render the form template to display the add transaction form if the request method is GET
    return render_template("form.html")

# Update operation: Route to display and process edit transaction form
@app.route("/edit/<int:transaction_id>", methods=["GET", "POST"])
def edit_transaction(transaction_id):
    if request.method == 'POST':
        # Extract the updated values from the form fields
        date = request.form['date']
        amount = float(request.form['amount'])

        # Find the transaction with the matching ID and update its values
        for transaction in transactions:
            if transaction['id'] == transaction_id:
                transaction['date'] = date       # Update the 'date' field of the transaction
                transaction['amount'] = amount   # Update the 'amount' field of the transaction
                break                            # Exit the loop once the transaction is found and updated

        # Redirect to the transactions list page after updating the transaction
        return redirect(url_for("get_transactions"))

    # Find the transaction with the matching ID and render the edit form if the request method is GET
    for transaction in transactions:
        if transaction['id'] == transaction_id:
            # Render the edit form template and pass the transaction to be edited
            return render_template("edit.html", transaction=transaction)

# Delete operation: Route to delete a transaction
@app.route("/delete/<int:transaction_id>")
def delete_transaction(transaction_id):
    # Find the transaction with the matching ID and remove it from the list
    for transaction in transactions:
        if transaction['id'] == transaction_id:
            transactions.remove(transaction)  # Remove the transaction from the transactions list
            break                            # Exit the loop once the transaction is found and removed

    # Redirect to the transactions list page after deleting the transaction
    return redirect(url_for("get_transactions"))

@app.route("/search", methods=["GET", "POST"])
def search_transactions():
    """
    Handles transaction filtering based on amount range.
    GET: Displays the search form (search.html).
    POST: Processes the form data and displays filtered results (transactions.html).
    """
    if request.method == 'POST':
        try:
            # 1. Retrieve and convert minimum and maximum amount values
            min_amount = float(request.form['min_amount'])
            max_amount = float(request.form['max_amount'])
        except ValueError:
            # Handle case where conversion to float fails (e.g., empty or invalid input)
            # For simplicity, we'll default to showing all transactions or a relevant message.
            # In a real app, you'd show an error message.
            return render_template("transactions.html", transactions=transactions, error="Invalid amount input.")

        # 2. Filter the transactions list using a list comprehension
        filtered_transactions = [
            t for t in transactions 
            if min_amount <= t['amount'] <= max_amount
        ]

        # 3. Pass the filtered list to the transactions.html template
        return render_template("transactions.html", transactions=filtered_transactions)

    # If the request method is GET, render the search form template
    return render_template("search.html")

@app.route("/balance")
def total_balance():
    """
    Calculates and returns the total balance as a simple string.
    """
    # 1. Calculate the total balance
    balance = sum(t['amount'] for t in transactions)
    
    # 2. Return the total balance as a formatted string
    return f"Total Balance: {balance:.2f}"

# Run the Flask application
if __name__ == "__main__":
    app.run(debug=True)
```
