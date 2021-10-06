# Testing



You can run your answers through mongo directly by running `mongo movies` to open the database and then entering your query directly:

```text
$ mongo movies
db.replace_with_a_collection.aggregate([ ... your query ... ])
```

This can help you catch any syntax errors in your queries. Alternatively you can run a query directly through the testing script by pasting in your query into the appropriate file \(we'll use `query/q0.js` as an example\) and running `python3 test.py q0 --view`. This will print up to ten of the query's results.

You can request more results with the `--batch_size` flag \(i.e. `python3 test.py q0 --view --batch_size 20` will give the first twenty results\). 

If you find yourself dealing with large, hard to read documents, you can view a formatted version of the first document by using `--format` instead of `--view`. For example, using the provided query from the [Building your first query](your-tasks.md#building-your-first-query) section of the spec:

```text
$ python3 test.py q0 --format
Showing formatted first document of the query
{
    "min_rating": 2,
    "max_rating": 5,
    "title": "Jurassic Park",
    "num_ratings": 48
}
```

To run a test, from within the `fa20-proj6-yourname` directory:

```text
$ python3 test.py     # This runs all of the tests
$ python3 test.py 3ii # This would run tests for only q3ii
```

## Format Matching

Before we run a full test on your output, we check that the format of your output matches what we're expecting. Format in this context means that all the field names we expect are there, there are no extra field names, and that the types corresponding to the field names match. Here's an example of some mismatched format for `diffs/q2i.diff`

```text
EXTRA FIELDS
- foo

MISMATCHED TYPES
- mismatch on field `movieId`:
    - expected type: `number` (example: `63`)
    - actual type: `null`, (example: `null`)

FORMAT MISMATCH
- Example of expected document:
{
    "movieId": 63
}

- Your document:
{
    "foo": "bar",
    "movieId": null
}
```

The above output tells you two things:

* One of your documents had an extra field. In this case, looking at the "Your document" section, your output had an extra field called "foo"
* One of your documents has a mismatched type for one of its fields. In this case, look at the "Your document" section, your output had a value of type null for the field "movieId", when it should have been a number.

## Diffs

If you pass the format check, we'll run a diff against your query and the expected output. Become familiar with the UNIX [diff](http://en.wikipedia.org/wiki/Diff) format, if you're not already, because our tests saves a simplified diff for any query executions that don't match in `diffs/`. As an example, the following output for `diffs/q2ii.diff:`:

```text
+ {"_id": "only", "count": 521}
  {"_id": "just", "count": 481}
- {"_id": "about", "count": 535}
```

This indicates that: 

* your output has an extra document `{"_id": "about", "count": 535}` \(the `-` at the beginning means the expected output _doesn't_ include this line but your output has it\)
* your output is missing the document `{"_id": "only", "count": 521}` \(the plus at the beginning means the expected output _does_ include those lines but your output is missing it\). 
* If there is neither a `+` nor `-` at the beginning then it means that the line is in both your output and the expected output \(your output is correct for that line\).

If you care to look at the query outputs directly, ours are located in the `expected_output` directory. Your output should be located in your solution's `your_output` directory once you run the tests. 

## Reformatting

When we generate the diffs we'll be doing some basic reformatting to reorder things that don't have an inherent order to make sure that the results are consistent with each other. These include:

* field names will be rearranged to be in alphabetical order
* arrays when we don't ask for an explicit order to them

