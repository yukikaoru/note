---
title: "conftest.pyを小分けにする"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["pytest"]
published: true
---

# conftest.pyを小分けにする

`conftest.py`に定義するフィクスチャも増えてくると次のようにファイルを分割したくなる。\
次のように単純にフィクスチャを分割したとする。

```
tests
|-- conftest.py
`-- fixtures
  |-- module1.py
  `-- module2.py
```

## インポートによる問題

```python
# conftest.py
from .fixtures.module1 import *
from .fixtures.module2 import *
```

こうすることで分けたモジュールから定義したフィクスチャを持ってこれるが、アスタリスクでインポートすると望んでいないものもインポートされてしまうので避けたい。\
そもそもアスタリスクによるインポートはコード検証ツールに引っかかる可能性が高い。

## pytest_pluginsによるロード

[pytest_plugins](https://pytest.org/en/7.1.x/how-to/plugins.html?highlight=hooks#requiring-loading-plugins-in-a-test-module-or-conftest-file)は元々はpytestのプラグイン作成で利用するフックなのだが、これによる解決もできる。

```python
pytest_plugins = (
    "tests.fixtures.module1",
    "tests.fixtures.module2",
)
```

## サブパッケージで分ける

公式ドキュメントにドストライクな[conftest.py: sharing fixtures across multiple files](https://docs.pytest.org/en/7.1.x/reference/fixtures.html?highlight=plugins#conftest-py-sharing-fixtures-across-multiple-files)という箇所がある。

これは各サブパッケージに`conftest.py`を配置すると、そのパッケージのテストが実行される際に利用されるという仕組みによるもの。

```
tests
|-- conftest.py
|-- module1
|  |-- conftest.py
|  `-- test_module1.py
`-- module2
   |-- conftest.py
   `-- test_module2.py
```

こういった構成の場合、`module1.test_module1`を実行した時のconftestは`tests.module1.conftest < tests.conftest`となる。
