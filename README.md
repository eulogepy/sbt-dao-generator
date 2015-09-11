# sbt-dao-generator

## プラグインのビルド方法

```sh
$ sbt clean package
```

## プラグインの利用方法

ローカルリポジトリにプラグインをインストールします。

```sh
$ sbt clean publish-local
```

project/plugins.sbtに以下のエントリを追加してください。

```scala
addSbtPlugin("jp.co.septeni-original" % "sbt-dao-generator" % "1.0.0-SNAPSHOT")
```

## プラグインの設定方法

以下を参考にbuild.sbtに設定を行ってください。

```scala
// JDBCのドライバークラス名を指定します(必須)
driverClassName in generator := "org.h2.Driver"

// JDBCの接続URLを指定します(必須)
jdbcUrl in generator := "jdbc:h2:file:./target/test"

// JDBCの接続ユーザ名を指定します(必須)
jdbcUser in generator := "sa"

// JDBCの接続ユーザのパスワードを指定します(必須)
jdbcPassword in generator := ""

// スキーマ名を指定できます(任意。デフォルトは以下)
schemaName in generator := None,

// カラム型名をどのクラスにマッピングするかを決める関数を記述します(必須)
typeNameMapper in generator := {
  case "INTEGER" => "Int"
  case "VARCHAR" => "String"
  case "BOOLEAN" => "Boolean"
  case "DATE" | "TIMESTAMP" => "java.util.Date"
  case "DECIMAL" => "BigDecimal"
}

// 出力対象のテーブルをフィルターする関数を記述できます(任意。デフォルトはすべてのテーブルが対象になります)
tableNameFilter in generator := { tableName: String => tableName.toUpperCase != "SCHEMA_VERSION"}

// テンプレートファイルを配置するディレクトリを指定できます(任意。デフォルトは以下)
templateDirectory in generator := baseDirectory.value / "templates"

// テンプレートファイル名を指定できます(任意。デフォルトは以下)
templateName in generator := "template.ftl"

// ソースコードを出力するディレクトリを指定できます(任意。デフォルトは以下。target/scala-2.xx/src_managed/)
outputDirectory in generator := sourceManaged.value / "main"

// テーブル名からモデルクラス名にマッピングする関数を記述できます(任意。デフォルトは以下)
modelNameMapper in generator := { tableName: String =>
    StringUtil.camelize(tableName)
}

// カラム名からプロパティ名にマッピングする関数を記述できます(任意。デフォルトは以下)
propertyNameMapper in generator := { columnName: String =>
    StringUtil.decapitalize(StringUtil.camelize(columnName))
}
```

## テンプレートファイル

templateName in generatorで指定されたテンプレートファイルを適宜編集してください。テンプレートエンジンは[FreeMarker](http://freemarker.org/)となります。

```
case class ${name}(
<#list columns as column>
<#if column.nullable>
${column.name}: Option[${column.typeName}]<#if column_has_next>,</#if>
<#else>
${column.name}: ${column.typeName}<#if column_has_next>,</#if>
</#if>
</#list>
) {

}
```

## コード生成

```sh
$ sbt generator::generate
<snip>
[info] tableName = DEPT, generate file = /Users/sbt-user/myproject/target/scala-2.10/src_managed/Dept.scala
[info] tableName = EMP, generate file = /Users/sbt-user/myproject/target/scala-2.10/src_managed/Emp.scala
[success] Total time: 0 s, completed 2015/06/24 18:17:20
```

## 開発者向け

### 単体テスト方法

プロジェクト直下に配置されたtest.mv.dbを使って単体テストを行います(test.mv.dbはcreate_table.sqlによって手動で作られたデータベースファイルです)。

```sh
$ sbt clean test
```

### プラグインの動作テスト方法

#### scriptedでテストを行う

Scripted Test Frameworkを使ったテストします。詳しくは[こちら](http://eed3si9n.com/ja/testing-sbt-plugins)を参考にしてください。
なお、生成されたソースコードは確認できません。

```
$ sbt clean scripted 
[info] Loading project definition from /Users/sbt-user/sbt-dao-generator/project
<snip>
Running sbt-dao-generator / simple
[info] Getting org.scala-sbt sbt 0.13.8 ...
[info] :: retrieving :: org.scala-sbt#boot-app
[info] 	confs: [default]
[info] 	52 artifacts copied, 0 already retrieved (17674kB/132ms)
[info] Getting Scala 2.10.4 (for sbt)...
[info] :: retrieving :: org.scala-sbt#boot-scala
[info] 	confs: [default]
[info] 	5 artifacts copied, 0 already retrieved (24459kB/78ms)
[info] [info] Loading project definition from /private/var/folders/tw/2_9djq693wj1889trb760g6w0000gn/T/sbt_9548a4f7/simple/project
[info] [info] Set current project to simple (in build file:/private/var/folders/tw/2_9djq693wj1889trb760g6w0000gn/T/sbt_9548a4f7/simple/)
[info] [info] Updating {file:/private/var/folders/tw/2_9djq693wj1889trb760g6w0000gn/T/sbt_9548a4f7/simple/}simple...
[info] [info] Resolving org.scala-lang#scala-library;2.10.4 ...
       [info] Resolving com.h2database#h2;1.4.187 ...
       [info] Resolving org.scala-lang#scala-compiler;2.10.4 ...
       [info] Resolving org.scala-lang#scala-reflect;2.10.4 ...
       [info] Resolving org.scala-lang#jline;2.10.4 ...
       [info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] [info] Done updating.
[info] [info] Flyway 3.2.1 by Boxfuse
[info] [info] Database: jdbc:h2:file:./target/test (H2 1.4)
[info] [info] Validated 1 migration (execution time 00:00.021s)
[info] [info] Current version of schema "PUBLIC": 1
[info] [info] Schema "PUBLIC" is up to date. No migration necessary.
[info] [success] Total time: 0 s, completed 2015/06/26 12:54:49
[info] [info] tableName = DEPT, generate file = /private/var/folders/tw/2_9djq693wj1889trb760g6w0000gn/T/sbt_9548a4f7/simple/target/scala-2.10/src_managed/Dept.scala
[info] [info] tableName = EMP, generate file = /private/var/folders/tw/2_9djq693wj1889trb760g6w0000gn/T/sbt_9548a4f7/simple/target/scala-2.10/src_managed/Emp.scala
[info] [success] Total time: 0 s, completed 2015/06/26 12:54:50
[info] + sbt-dao-generator / simple
[success] Total time: 23 s, completed 2015/06/26 12:54:50
```

#### 実際にプラグインをロードして実行する方法

実際に生成されたソースコード確認できます。

```sh
$ sbt -Dplugin.version=1.0.0-SNAPSHOT                                                                                                         
[info] Loading project definition from /Users/sbt-user/sbt-dao-generator/src/sbt-test/sbt-dao-generator/simple/project
[info] Set current project to simple (in build file:/Users/sbt-user/sbt-dao-generator/src/sbt-test/sbt-dao-generator/simple/)
> flywayMigrate
[info] Flyway 3.2.1 by Boxfuse
[info] Database: jdbc:h2:file:./target/test (H2 1.4)
[info] Validated 1 migration (execution time 00:00.020s)
[info] Current version of schema "PUBLIC": 1
[info] Schema "PUBLIC" is up to date. No migration necessary.
[success] Total time: 0 s, completed 2015/06/24 18:17:12
> generator::generate
[info] tableName = DEPT, generate file = /Users/sbt-user/sbt-dao-generator/src/sbt-test/sbt-dao-generator/simple/target/scala-2.10/src_managed/Dept.scala
[info] tableName = EMP, generate file = /Users/sbt-user/sbt-dao-generator/src/sbt-test/sbt-dao-generator/simple/target/scala-2.10/src_managed/Emp.scala
[success] Total time: 0 s, completed 2015/06/24 18:17:20
```