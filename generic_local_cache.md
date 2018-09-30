# Generic Local Cache

## Task:
You're tasked with writing a spec for a generic local cache with the following property: If the
cache is asked for a key that it doesn't contain, it should fetch the data using an externally
provided function that reads the data from another source (database or similar).
What features do you think such a cache should offer? How, in general lines, would you
implement it?
