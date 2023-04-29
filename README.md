<h1>Using Relationalize to transfer Data from MongoDB to MySQL.</h1>
Relationalize is a python library that helps transform collections of documents in JSON objects format into sets of normalized tables that can be loaded in a relational database. This normalized tables are then converted to CSV files which in turn are easily written into a relational database. 
      //example of json objects
      {'Address': 'Sears Streer, NZ',
      'Age': '42',
      'Name': 'Raj Kumar',
      '_id': ObjectId('64307ef302a0320de41984ea')}
      {'_id': ObjectId('6430add2cd83c136312f3ca0'),
      'author': 'Martin',
      'contributors': ['Aldren', 'Geir Arne', 'Jaya', 'Joanna', 'Mike'],
      'title': 'Beautiful Soup: Build a Web Scraper With Python',
      'url': 'https://realpython.com/beautiful-soup-web-scraper-python/'}

In order to transfer the files from non-relational database to relational database, one must overcome two main challenges in order to make it happen. The two main challenges are listed below.
<ol type = 1>
  <li>Sparse columns ("contributors", "name", "author", "age", field)</li>
  <li>Arrays ("Contributors", field)</li>
</ol>

Relationalize provides solution to these challenges, it is faster, portable, and has less limitations compared to some other methods.

<h2>Method:</h2>
Relationalize repeatedly navigates the JSON object, splitting the documents in a collection whenever it encounters an array object, then it provides connection between the objects i.e the separated arrays and the rest of the data.

The relationalize class is provided with a function that determines where the transformed data is written. Additionally, any nested objects that is output by relationalize becomes a flattened JSON object.

This package also provides a "Schema class" which generates a schema for a collection of flattened JSON objects. This schema can also be used to handle type ambiguity and to also generate SQL DDL.

When processing JSON objects in a collection, the schema is not known unless we provide a function that will enable relationalize class to generate schema and store the new collections it will eventually create. it takes a function (identifier: str) -> TextIO as an argument then creates an output. The function "(identifier: str) -> TextIO" is used to create an output.

It also takes an optional function which called whenever an object is written to a file created by the (identifier: str) -> TextIO function. This method can be used to generate the schema as the objects are encountered. This reduces the number of iterations over the JSON objects.

  
    For example:
    schemas: Dict[str, Schema] = {}
    def on_object_write(schema: str, object: dic):
	    if schema not in schemas:
		    schemas[schema] = Schema()
	      schemas[schema].read_object(object)
    
    def create_iterator(filename) -> Generator[Dict[str, Any], None, None]:
      with open(filename, "r") as infile:
        for line in infile:
            yield json.loads(line)

    with Relationalize('object_name', on_object_write=on_object_write) as r:
          r.relationalize(create_iterator)

After relationalizing the JSON objects and the schemas have been generated, the objects are then converted into the final JSON object which is then loaded into a relational database. convert_object will break out any ambiguosly typed columns into seperate columns. 

The second document in the collection would output the following three documents after being processed by relationalized and convert_object:

    #employees
    {
    '_id': ObjectId('6430add2cd83c136312f3ca0'),
    'author': 'Martin',
    'title': 'Beautiful Soup: Build a Web Scraper With Python',
    'url': 'https://realpython.com/beautiful-soup-web-scraper-python/'}

    {'_id': ObjectId('6430add2cd83c136312f3ca0'),
    'author': 'Martin',
    'contributors': ['Aldren', 'Geir Arne', 'Jaya', 'Joanna', 'Mike'],
    'title': 'Beautiful Soup: Build a Web Scraper With Python',
    'url': 'https://realpython.com/beautiful-soup-web-scraper-python/'
    'contributors': 'R_243787c5dbc14203995f3cecfbf310b0'}

    #employees_contributors
    {
      'contributors__rid_': R_243787c5dbc14203995f3cecfbf310b0	,
      'contributors__index_': '0',
      'contributors_val_': 'Aldren' }
    {
      'contributors__rid_': 'R_243787c5dbc14203995f3cecfbf310b0'),
      'contributors__index_': '1',
      'contributors_val_': 'Geir Arne' }
<h2>Conclusion</h2>
After relationalize splits the files into different groups, it is read and converted to csv file extension which are then used to write the data into the relational database.
