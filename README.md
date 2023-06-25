__MarketBasketAnalysis__
-------------------------------------
Market Basket Analysis (MBA) is a data mining technique used to reveal products that are likely to be purchased together by analyzing dataset. Each rows corresponds to the item bought by one customer in one invoice. E.g. the rule {turkey} -> {avocado} found in the sales data of a supermarket would indicate that if a customer buys turkey, they are likely to also buy avocado. In this repository, the Apriori algorithm is used to perform Market Basket Analysis. <br>

### Converting DataFrame to Array of Strings
First, it is necessary to convert the DataFrame into array of strings. This can be done using the following code snippet:
``` python
transaction = []

for i in range(0,7501):
    transaction.append([str(df.values[i,j]) for j in range(0,20)])
transaction = np.array(transaction)
```
In the above code, a transaction list is initialized. Then, a loop iterates over the range from 0 to 7500 (inclusive). Within each iteration, a list comprehension is used to convert the values from the DataFrame df into strings. The converted values are appended to the transaction list. After the loop completes, the transaction list is converted into a NumPy array for further analysis.

### Remove Missing Values
After converting the DataFrame into an array, there might be missing values represented as `NaN`. It is important to remove these missing values before performing Market Basket Analysis. 

``` python
transaction_str = ' '.join(transaction.flatten())
transaction_str = transaction_str.replace('nan','')
```

### Create word cloud
As a supplementary visual representation, word cloud can be used to visualize the popularity of products by making the most frequently used product appear larger compared with the other words around them.

``` python
from wordcloud import WordCloud

# Generate the word cloud
wordcloud = WordCloud(width=1000, height=1000, max_words=20).generate(transaction_str)

# Display the word cloud
plt.imshow(wordcloud)
plt.axis('off')
plt.title('Most popular items bought by Customers', fontsize = 20)
plt.show()
```
Here is the result:<br>
<p align="center">
  <img src="https://github.com/hhanarafifa/MarketBasketAnalysis/assets/103975566/aac16a43-77f4-4801-8562-77cc70057c0a">
</p>

### Display Top 10 Best-Selling products
Beside the word cloud, the top 10 best-selling products can also be illustrated on bar chart. Here is the code snippet:

``` python
# First, we need to flatten the array
transaction_flat = transaction.flatten()

# remove 'nan' values
filtered_str = transaction_flat[transaction_flat != 'nan']

# count the product frequency
product_count = Counter(filtered_str)

top_products = sorted(product_count.items(), key=lambda x: x[1], reverse=True)[:10]
top_products

# Extract product names and their respective frequencies
product_names = [product[0] for product in top_products]
frequencies = [product[1] for product in top_products]
plt.figure(figsize = (16,9))

plt.bar(product_names, frequencies, color = (0.2, 0.3, 0.5, 0.5))
plt.xlabel('Products', fontsize = 18)
plt.ylabel('Quantity Sold')
plt.title('Top 10 Best-Selling Products')
plt.xticks(rotation=90)
plt.show()
```
<p align="center">
  <img src="https://github.com/hhanarafifa/MarketBasketAnalysis/assets/103975566/7952ea12-7989-49c3-ae34-cd8c1bb04ac4">
</p>
<br> As can be seen from the word cloud, mineral water is the most frequently bought product. Eggs are in second place, while spaghetti, French fries, and chocolate occupy the next positions. Green tea and milk rank sixth and seventh, respectively, followed by ground beef, frozen vegetables, and pancakes.

### Perform Market Basket Analysis
Market Basket Analysis are performed using the code as follows:
``` python
# Perform Market Basket Analysis3

transaction = list(transaction)
transaction

rules = apriori(transaction, min_support = 0.002, min_confidence = 0.05, min_lift = 3, min_length=2)
result = list(rules)
result
```
![Screen Shot 2023-06-25 at 19 59 33](https://github.com/hhanarafifa/MarketBasketAnalysis/assets/103975566/998600fa-fba1-41da-b7ca-3a366ef3444d)
The outputs are pretty indecipherable. Therefore, new DataFrame is created to facilitate analysis and interpretation.
<br>
```python
def inspect (result):
    lhs = [tuple(i[2][0][0])[0]for i in result]
    rhs = [tuple(i[2][0][1])[0]for i in result]
    support = [i[1] for i in result]
    confidences = [i[2][0][2] for i in result]
    lifts = [i[2][0][3] for i in result]
    return list(zip(lhs,rhs,support,confidences, lifts))
results = pd.DataFrame(inspect(result),columns = ['Subset 1', 'Subset 2', 'support', 'confidences', 'lifts'])
results = results.sort_values(by=['lifts'], ascending=False)
results = results.drop_duplicates(keep='first')

results[:10]
```
<p align="center"> 
  <img src=https://github.com/hhanarafifa/MarketBasketAnalysis/assets/103975566/34ae69b8-21db-4f7e-b662-9bed63207238>
</p>
