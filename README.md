### 编译目的
`Maven`仓库中的`IKAnalyzer`版本太老，不能用于`lucene5.0`以上的版本。故重新编译，使其可以使用`lucene6.6.0`。

### 源码下载

[地址](http://git.oschina.net/wltea/IK-Analyzer-2012FF)

### 步骤

#### 编写`pom`文件

pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.janeluo</groupId>
    <artifactId>ikanalyzer</artifactId>
    <version>6.6.0</version>
    <packaging>jar</packaging>

    <name>ikanalyzer</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>4.12</junit.version>
        <lucene.version>6.6.0</lucene.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>

        <!-- lucene -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-core</artifactId>
            <version>${lucene.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-analyzers-common</artifactId>
            <version>${lucene.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-queryparser</artifactId>
            <version>${lucene.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-queries</artifactId>
            <version>${lucene.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-analyzers-smartcn</artifactId>
            <version>${lucene.version}</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>${project.name}</finalName>
        <resources>
            <resource>
                <filtering>false</filtering>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.xml</include>
                    <include>**/*.dic</include>
                </includes>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.4</version>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 修改`IKTokenizer.java`

删除构造方法中的`super(in)`

#### 修改`IKAnalyzer.java`

```
/**
 * 重载Analyzer接口，构造分词组件
 */
@Override
protected TokenStreamComponents createComponents(String fieldName) {
    Reader in = new BufferedReader(new StringReader(fieldName));
	Tokenizer _IKTokenizer = new IKTokenizer(in , this.useSmart());
	return new TokenStreamComponents(_IKTokenizer);
}
```

#### 修改`IKQueryExpressionParser.java`

```
private Query toBooleanQuery(Element op){
	if(this.querys.size() == 0){
		return null;
	}
	
	BooleanQuery.Builder resultQuery = new BooleanQuery.Builder();

	if(this.querys.size() == 1){
		return this.querys.get(0);
	}
	
	Query q2 = this.querys.pop();
	Query q1 = this.querys.pop();
	if('&' == op.type){
		if(q1 != null){
			if(q1 instanceof BooleanQuery){
				List<BooleanClause> clauses = ((BooleanQuery)q1).clauses();
				if(clauses.size() > 0 
						&& clauses.get(0).getOccur() == Occur.MUST){
					for(BooleanClause c : clauses){
						resultQuery.add(c);
					}					
				}else{
					resultQuery.add(q1,Occur.MUST);
				}

			}else{
				//q1 instanceof TermQuery 
				//q1 instanceof TermRangeQuery 
				//q1 instanceof PhraseQuery
				//others
				resultQuery.add(q1,Occur.MUST);
			}
		}
		
		if(q2 != null){
			if(q2 instanceof BooleanQuery){
				List<BooleanClause> clauses = ((BooleanQuery)q2).clauses();
				if(clauses.size() > 0 
						&& clauses.get(0).getOccur() == Occur.MUST){
					for(BooleanClause c : clauses){
						resultQuery.add(c);
					}					
				}else{
					resultQuery.add(q2,Occur.MUST);
				}
				
			}else{
				//q1 instanceof TermQuery 
				//q1 instanceof TermRangeQuery 
				//q1 instanceof PhraseQuery
				//others
				resultQuery.add(q2,Occur.MUST);
			}
		}
		
	}else if('|' == op.type){
		if(q1 != null){
			if(q1 instanceof BooleanQuery){
				List<BooleanClause> clauses = ((BooleanQuery)q1).clauses();
				if(clauses.size() > 0 
						&& clauses.get(0).getOccur() == Occur.SHOULD){
					for(BooleanClause c : clauses){
						resultQuery.add(c);
					}					
				}else{
					resultQuery.add(q1,Occur.SHOULD);
				}
				
			}else{
				//q1 instanceof TermQuery 
				//q1 instanceof TermRangeQuery 
				//q1 instanceof PhraseQuery
				//others
				resultQuery.add(q1,Occur.SHOULD);
			}
		}
		
		if(q2 != null){
			if(q2 instanceof BooleanQuery){
				List<BooleanClause> clauses = ((BooleanQuery)q2).clauses();
				if(clauses.size() > 0 
						&& clauses.get(0).getOccur() == Occur.SHOULD){
					for(BooleanClause c : clauses){
						resultQuery.add(c);
					}					
				}else{
					resultQuery.add(q2,Occur.SHOULD);
				}
			}else{
				//q2 instanceof TermQuery 
				//q2 instanceof TermRangeQuery 
				//q2 instanceof PhraseQuery
				//others
				resultQuery.add(q2,Occur.SHOULD);
				
			}
		}
		
	}else if('-' == op.type){
		if(q1 == null || q2 == null){
			throw new IllegalStateException("表达式异常：SubQuery 个数不匹配");
		}
		
		if(q1 instanceof BooleanQuery){
			List<BooleanClause> clauses = ((BooleanQuery)q1).clauses();
			if(clauses.size() > 0){
				for(BooleanClause c : clauses){
					resultQuery.add(c);
				}					
			}else{
				resultQuery.add(q1,Occur.MUST);
			}

		}else{
			//q1 instanceof TermQuery 
			//q1 instanceof TermRangeQuery 
			//q1 instanceof PhraseQuery
			//others
			resultQuery.add(q1,Occur.MUST);
		}				
		
		resultQuery.add(q2,Occur.MUST_NOT);
	}
	return resultQuery.build();
}
```

#### 去掉所有的`Version.LUCENE_40`




