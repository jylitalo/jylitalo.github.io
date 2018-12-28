---
layout: post
title: "Doctest in Python"
status: publish
date: 2015-09-14 12:00
---
My Assets on Octopress script needed couple modifications and I felt that it was good platform for doing unit tests. Python has nice test framework called [doctest](http://docs.python.org/3/library/doctest.html), which allows you to insert unit test cases as comments in your python code. My current tests are done for methods that don't recover mockups (for things like file system access) and simply return strings, lists and such to caller.

In following example, I have method called `_extract_from_markdown`, which gets one line of text from markdown file as string. It will try to find all possible references to files under /assets/ or /images/ and return those as list of strings. Comment lines that starts with '>>>' are method calls and lines below them (starting with [) are expected return values from method.

```
def _extract_from_markdown(line):
  """                                                                                                                        
  >>> AssetsFinder._extract_from_markdown("foobar")
  []
  >>> AssetsFinder._extract_from_markdown('[foobar](/images/link.jpg)')
  ['/images/link.jpg']
  >>> AssetsFinder._extract_from_markdown('[foobar](/images/link.jpg "foobar")')
  ['/images/link.jpg']
  >>> AssetsFinder._extract_from_markdown('{\% img /images/something.jpg "foobar"}')
  ['/images/something.jpg']
  >>> AssetsFinder._extract_from_markdown('[![#48/2012: Rautatientori](/images/2012/11/IMG_0156_t.jpg "#48/2012: Rautatientori")](/images/2012/11/IMG_0156_l.jpg "#48/2012: Rautatientori")')
  ['/images/2012/11/IMG_0156_t.jpg', '/images/2012/11/IMG_0156_l.jpg']                                                     
  """
  links = []
  for key in ["images","assets"]:
    for field in line.split("(/" + key)[1:]:
      field = "/%s%s" % (key,field[:field.find(')')])
      if ' "' in field: field = field[:field.find(' "')
      links.append(field)
    for c in [' ','"',"'"]:
      for field in line.split("%s/%s" % (c,key))[1:]:
        field = "/" + key + field[:field.find(c,1)]
        links.append(field)
  return links
```

As slight more advanced tests, find_source_dir method will throw AssertionException, if it is unable to guess root directory in your Octoress directory tree. First two tests are successful cases, while the third one (about /bound/to/fail directory) will throw exception.

```
def find_source_dir(dir=os.getcwd()):
  """                                                                                                                        
  >>> Octopress.find_source_dir("/tmp/octo/source/_posts")                                                                   
  '/tmp/octo/source'                                                                                                         
  >>> Octopress.find_source_dir("/tmp/octo/source")                                                                          
  '/tmp/octo/source'                                                                                                         
  >>> Octopress.find_source_dir("/bound/to/fail")                                                                            
  Traceback (most recent call last):                                                                                         
  ...                                                                                                                        
  AssertionError: unable to determine source directory from /bound/to/fail                                                   
  """
  if dir.endswith("/source"): return dir
  if dir.endswith("/source/_posts"): return dir[:dir.rfind('/')]
  if os.access(dir + "/config.rg", os.R_OK) and os.access(dir + "/source",os.F_OK):
    return dir + "/source"
  raise AssertionError("unable to determine source directory from " + dir)
```

Now that we have written tests into our python script, we should execute them and check results. Short and sweet version is to run `python -m doctest assets_on_octopress.py`, but it will only report failed tests cases. To get little more complete report out of it, we have to call `python -m doctest -v assets_on_octopress.py`.

Verbose version will first go through all test cases in following manner:

```
Trying:
    AssetsFinder._extract_from_markdown("foobar")
Expecting:
    []
ok
Trying:
    AssetsFinder._extract_from_markdown('[foobar](/images/link.jpg)')
Expecting:
    ['/images/link.jpg']
ok
Trying:
    AssetsFinder._extract_from_markdown('[foobar](/images/link.jpg "foobar")')
Expecting:
    ['/images/link.jpg']
ok
...
```

And based on that, it will create nice summary about what was tested and what areas still need more work.

```
20 items had no tests:
    assets_on_octopress
    assets_on_octopress.AssetsFinder
    assets_on_octopress.AssetsFinder.__init__
    assets_on_octopress.AssetsFinder._find_markdown_files
    assets_on_octopress.AssetsFinder._scan_tree
    assets_on_octopress.AssetsFinder._validate_found
    assets_on_octopress.AssetsFinder._validate_missing
    assets_on_octopress.AssetsFinder._validate_waste
    assets_on_octopress.AssetsFinder.dir
    assets_on_octopress.AssetsFinder.scan
    assets_on_octopress.AssetsFinder.validate
    assets_on_octopress.AssetsFixer
    assets_on_octopress.AssetsFixer.__init__
    assets_on_octopress.AssetsFixer._original_image
    assets_on_octopress.AssetsFixer._resize_image
    assets_on_octopress.AssetsFixer._validate_found
    assets_on_octopress.AssetsFixer._validate_missing
    assets_on_octopress.AssetsFixer._validate_waste
    assets_on_octopress.Octopress
    assets_on_octopress.Octopress.head
4 items passed all tests:
   5 tests in assets_on_octopress.AssetsFinder._extract_from_markdown
   5 tests in assets_on_octopress.AssetsFinder._ignore
   6 tests in assets_on_octopress.AssetsFinder.get_url
   3 tests in assets_on_octopress.Octopress.find_source_dir
19 tests in 24 items.
19 passed and 0 failed.
Test passed.
```
