# Resource Query Language (RQL) parser for Java

Resource Query Language or RQL is a query language which provides a nice way of querying a data store, SQL database etc.
via a URI. For more details on RQL see
this [Sitepen blog article](https://www.sitepen.com/blog/2010/11/02/resource-query-language-a-query-language-for-the-web-nosql/)
by Kris Zyp and the Dojo Foundation's [Persevere RQL repository](https://github.com/persvr/rql) on Github. The short of
it is that you can write queries in URI's like this:

```
/books?author=in=(Desmond%20Bagley,Alistair%20Maclean)|genre=Thriller
```

I have written a Java parser for RQL since I couldn't find one and published it on the [Maven Central Repository](http://search.maven.org/). The
parser's logic is based on the Javascript parser from the Persevere project so it should work for most situations.

## Usage

The parser generates an ASTNode object representing the root node of an Abstract Syntax Tree (AST). Create a class that
implements ASTVisitor in order to traverse the tree.

```java
RQLParser parser = new RQLParser();
MyVisitor visitor = new MyVisitor();
ASTNode node = parser.parse("(name=jack|name=jill)&age>30");
Object result = node.accept(visitor);
```

## Examples

See
the [ListFilter class](https://github.com/jazdw/rql-parser/blob/master/src/test/java/net/jazdw/rql/parser/listfilter/ListFilter.java),
which is an ASTVisitor which filters a list of bean objects.

## Get it from Maven

```xml
<dependency>
  <groupId>net.jazdw</groupId>
  <artifactId>rql-parser</artifactId>
  <version>0.3.1</version>
</dependency>
```
