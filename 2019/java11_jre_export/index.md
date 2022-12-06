---
# moved from https://aoirint.hatenablog.com/entry/2019/02/06/131412
title: 'Java 11 JRE付きエクスポート'
date: '2019-02-06T13:14:12+09:00'
draft: false
channel: 技術ノート
category: Java
tags:
  - 'Java'
---
# Java 11 JRE付きエクスポート

あんまり分かってないけど、とりあえずJRE付きでエクスポートまではできた。

## Java 9以降のこと

- Oracle Java(JDK)のリリースモデルが変わった
- Public JRE（Oracleが無償配布してたJRE）がなくなった
- Oracle JDKの配布が有償のみになる（Java 11以降）

- [来月にはJava 10が登場し、9月にはJava 11が登場予定。新しいリリースモデルを採用した今後のJava、入手方法やサポート期間はこう変わる（OpenJDKに関する追記あり） － Publickey](https://www.publickey1.jp/blog/18/java_109java_11java.html)

## これまでの実行可能Jar

- Javaランタイム（JRE）を同梱しなくても、システムにインストールされたPublic JREで実行できた

## これから
- システムにJREがない
- 同様に実行できるようにするには、JREをアプリケーションに同梱する必要がある（？）
  - これまでも`rt.jar`がJREの同梱された形としてあったはず

## Jigsawモジュールシステム

Java 9以降導入されたJavaの依存関係システム。

依存関係は`module-info.java`ファイルに記述する。

```java
module sample.module {
    requires java.base; // 不要（省略可）
}
```

- [モジュールシステムを学ぶ - Qiita](https://qiita.com/opengl-8080/items/93c8e0cf58654d5f73cb)

## 実行可能Jarのエクスポート（Maven、JREなし）

Java 8まで、こんな感じだった。

### Maven pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.sample</groupId>
    <artifactId>Sample</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.1</version>

                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.example.sample.Main</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

## JREの生成（手動、module-info使用）

- [アプリケーション配布用に小さなJREを作る](https://blogs.osdn.jp/2018/03/26/jre-minimize.html)
- [Jigsaw 勉強メモ - Qiita](https://qiita.com/opengl-8080/items/1007c2b2543c2fe0d7d5)

コンパイルしたクラスファイルのディレクトリを指定

```shell
jlink --module-path './classes;JDK_DIRECTORY/jmods' --add-modules sample.module --output jre
```

Jarを指定

```shell
jlink --module-path './sample.jar;JDK_DIRECTORY/jmods' --add-modules sample.module --output jre
```

圧縮

`-c 2`

起動用スクリプトも生成してくれる。

`--launcher mycmd=sample.module/com.example.sample.Main`

OpenJDK 11.0.1 (Windows)で実行すると、dll, exe, ランチャーシェルスクリプト, bat他が生成された。OpenJDKの中身を絞って移してきてる？　複数OS想定する場合、JREはOS別に作る必要があるのか？　それとも別の方法があるのか。


## 実行可能Jarのエクスポート（Maven、JREあり、ランチャー）

上の処理を自動化してくれるプラグインを使う。

- [java - Is there a maven jigsaw jlink plugin? - Stack Overflow](https://stackoverflow.com/questions/44360572/is-there-a-maven-jigsaw-jlink-plugin)

ant-runの部分は`jlink`ツールが出力ディレクトリを上書きしてくれないので、`package`を実行するごとに削除させるため。手動で消すか`clean`すれば不要。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.sample</groupId>
    <artifactId>Sample</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>

    <build>
        <plugins>
            <!-- https://mvnrepository.com/artifact/org.moditect/moditect-maven-plugin -->
            <plugin>
                <groupId>org.moditect</groupId>
                <artifactId>moditect-maven-plugin</artifactId>
                <version>1.0.0.Beta2</version>
                <executions>
                    <execution>
                        <id>create-runtime-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>create-runtime-image</goal>
                        </goals>
                        <configuration>
                            <compression>2</compression>
                            <modulePath>
                                <path>${project.build.outputDirectory}</path>
                            </modulePath>
                            <modules>
                                <module>sample.module</module>
                            </modules>
                            <launcher>
                                <name>Sample</name>
                                <module>sample.module/com.example.sample.Main</module>
                            </launcher>
                            <outputDirectory>
                                ${project.build.directory}/jlink
                            </outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-antrun-plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.8</version>
                <executions>
                    <execution>
                        <phase>initialize</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                        <configuration>
                            <target>
                                <delete dir="${project.build.directory}/jlink" quiet="true" />
                            </target>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

ただ、この方法だとjarファイルが見当たらない。たぶんruntimeも自分のプログラムも混ざってどこかに入ってる。外部jarを混ぜずに読み込む方法もわからない。

## JREの生成（手動、Jar使用）

`module-info`を使わない方法（使わないでいいのかは分からないけど）。

`jdeps --list-deps JARファイル`で`module-info`のないJARの依存するモジュールリストが出力される。

このリストをもとに下のコマンドでJREを生成すれば混ざることはないはず。

```shell
jlink -c 2 -p 'JDK_DIRECTORY/jmods' -m モジュールカンマ区切りリスト --output jre
```

生成されたjava実行ファイルに対して`java -jar JARファイル`するスクリプトを書けば外にjarを置ける。

外部Jarライブラリをlibsディレクトリに置く、というのを次はやりたい。今まで通りクラスパスに追加して、JREだけ上の方法で生成する、実行はスクリプトから、は正解じゃない気がするので、どうすればいいのか。それから、Launch4j使いたい場合？

- [Using jlink to Build Java Runtimes for non-Modular Applications | by Simon Ritter | Azul Systems | Medium](https://medium.com/azulsystems/using-jlink-to-build-java-runtimes-for-non-modular-applications-9568c5e70ef4)
