# stylespec

serverspec のように、CSSの内容ををテストで宣言する [stylespec](https://github.com/MisumiRize/stylespec) というおもちゃを作ってみた。

```ruby
gem 'stylespec', :github => 'MisumiRize/stylespec'
```

Gemfile にリポジトリを直接指定して、`bundle install` で使えるようになる。

```ruby
require "stylespec"
include Stylespec::Helpers

RSpec.configuration.root = "/path/to/public_root/css"
```

spec/spec_helper.rb に設定を書き込んで、普通の RSpec と同じようにテストを書く。

```ruby
require "spec_helper"

before :all do
  RSpec.configuration.name = "style.css"
end

describe selector(".foo") do
  it { should have_style :padding, "10px" }
end
```

あとは普通に rake spec なり rspec なりで実行。

作ってみたわりにいまいち使いどころが出てこないけど、 CI ツールで回してたらテストがこけてデザイン崩れ発覚、みたいな応用ができるかもしれない。
