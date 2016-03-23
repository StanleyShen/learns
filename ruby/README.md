# Deep merge of Hash
In some case, we want to do deep merge for 2 hash. for example
```bash
$ cat a.yml
---
name:
  cf1: stanley
age:
   aa: 11

$ cat b.yml
---
age:
  bb: 22
stanley: test
```

And we want get result
```
$ cat aaxxxx.yml
---
name:
  cf1: stanley
age:
  aa: 11
  bb: 22
stanley: test
```

An workable solution is to add one deep_merge method to hash object:

```ruby
require 'yaml'

cf_yml=ARGV.shift
stub_yml=ARGV.shift

if cf_yml == nil || stub_yml == nil
  puts "Usage: ruby add_manifest.rb <cf maninest path> <stub yml path>"
  exit 0
end

class ::Hash
    def deep_merge(second)
        merger = proc { |key, v1, v2| Hash === v1 && Hash === v2 ? v1.merge(v2, &merger) : Array === v1 && Array === v2 ? v1 | v2 : [:undefined, nil, :nil].include?(v2) ? v1 : v2 }
        self.merge(second, &merger)
    end
end

cf=YAML.load_file(cf_yml)
stub = YAML.load_file('stub.yml')

result = cf.deep_merge(stub)
File.open(cf_yml, 'w') {|f| f.write result.to_yaml }
```
