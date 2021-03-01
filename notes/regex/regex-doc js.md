# regex in javascript

* str.split(re)
* str.match(re)
* str.replace(re,
* re.test(str)
* re.exec(str)

---

## replacing in javascript

``` js
'1 two three\n'.replace(/1/, 'one')
```

similar to:

``` bash
echo '1 two three' | sed 's/1/one/'
```

---
## match testing in javascript

``` js
if (/^-(h|-help)$/.test(process.argv[2])) {
  console.log('usage: ...')
}
```
---
## capturing in javascript

``` js
var m = /^hello (\S+)/.test('hello cat')
console.log(m[1]) // cat
```

---
## splitting in javascript

``` js
> 'one_two-three'.split(/[_-]/)
[ 'one', 'two', 'three' ]
```

---

## capture groups in javascript

``` js
var str = 'hey <cool> whatever'
var m = /<([^]+)>/.exec(str)
console.log(m[1]) // cool
```

``` js
var str = 'hey <cool> whatever'
console.log(str.replace(/<([^]+)>/,'MATHEMATICAL'))
```


#js #regex