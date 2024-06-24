# Dart

## Modify GC

Add promotion age for youngGC.

为dart Runtime新增一个feature：增加new obj到old obj的promotion age，目前dart的实现age为1

### 设计思路

1. 目前的dart采用survivor_end指针标记上个youngGC结束之后to space实际内存的结束位置，由此可以考虑增加多个1st_survivor_end, 2nd_survivor_end指针来标记多次gc结束之后的位置。但在进行youngGC时，一个root obj可能会有到不同age区obj的引用，这样计算新地址就会变得较为复杂，不适合频繁出发的youngGC需求。

2. 参考openjdk的设计，在每个obj的header（untaggedObject）中留出标志位来记录obj的promotion age值。

### 标志位选取

1. 32位的hashTag，因为占位多于是优先考虑hashtag标识位，但实际测试下来发现这个位数和dart内置的hash算法实现强相关，不建议修改。

2. 4位的sizeTag，因为大对象会直接分配到old区，那么sizeTag高位或许可以用作promotion age的标记位。但实际上4位的sizeTag对应的大小只有sizeTagVal * 16（因为dart的obj是双字对齐的）而实际大小超过240（(2^4-1) * 16）的对象size会通过classid对应的类信息中获取。由此不建议修改。

3. 4位的gcTag，和gc流程设计与性能相关，修改会引入额外的访问与更新机制，暂不考虑修改，后续可以考虑基于此设计一套新的gcTag位来满足更灵活的age设计。
- kOldAndNotMarkedBit = 2,      // Incremental barrier target.
- kNewBit = 3,                  // Generational barrier target.
- kOldBit = 4,                  // Incremental barrier source.
- kOldAndNotRememberedBit = 5,  // Generational barrier source.

最后，采用1位的reservedTag，保留位目前没有被启用，可以用作age的简单扩展。
 
## An approachable, portable, and productive language for high-quality apps on any platform

Dart is:

  * **Approachable**:
  Develop with a strongly typed programming language that is consistent,
  concise, and offers modern language features like null safety and patterns.

  * **Portable**:
  Compile to ARM, x64, or RISC-V machine code for mobile, desktop, and backend.
  Compile to JavaScript or WebAssembly for the web.

  * **Productive**:
  Make changes iteratively: use hot reload to see the result instantly in your running app.
  Diagnose app issues using [DevTools](https://dart.dev/tools/dart-devtools).

Dart's flexible compiler technology lets you run Dart code in different ways,
depending on your target platform and goals:

  * **Dart Native**: For programs targeting devices (mobile, desktop, server, and more),
  Dart Native includes both a Dart VM with JIT (just-in-time) compilation and an
  AOT (ahead-of-time) compiler for producing machine code.

  * **Dart Web**: For programs targeting the web, Dart Web includes both a development time
  compiler (dartdevc) and a production time compiler (dart2js).  

![Dart platforms illustration](docs/assets/Dart-platforms.svg)

## License & patents

Dart is free and open source.

See [LICENSE][license] and [PATENT_GRANT][patent_grant].

## Using Dart

Visit [dart.dev][website] to learn more about the
[language][lang], [tools][tools], and to find
[codelabs][codelabs].

Browse [pub.dev][pubsite] for more packages and libraries contributed
by the community and the Dart team.

Our API reference documentation is published at [api.dart.dev](https://api.dart.dev),
based on the stable release. (We also publish docs from our 
[beta](https://api.dart.dev/beta) and [dev](https://api.dart.dev/dev) channels,
as well as from the [primary development branch](https://api.dart.dev/be)).

## Building Dart

If you want to build Dart yourself, here is a guide to
[getting the source, preparing your machine to build the SDK, and
building](https://github.com/dart-lang/sdk/wiki/Building).

There are more documents on our [wiki](https://github.com/dart-lang/sdk/wiki).

## Contributing to Dart

The easiest way to contribute to Dart is to [file issues][dartbug].

You can also contribute patches, as described in [Contributing][contrib].

[website]: https://dart.dev
[license]: https://github.com/dart-lang/sdk/blob/main/LICENSE
[repo]: https://github.com/dart-lang/sdk
[lang]: https://dart.dev/guides/language/language-tour
[tools]: https://dart.dev/tools
[codelabs]: https://dart.dev/codelabs
[dartbug]: http://dartbug.com
[contrib]: https://github.com/dart-lang/sdk/blob/main/CONTRIBUTING.md
[pubsite]: https://pub.dev
[patent_grant]: https://github.com/dart-lang/sdk/blob/main/PATENT_GRANT
