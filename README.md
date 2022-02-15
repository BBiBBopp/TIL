# TIL

# 22년 2월 16일
- MyBatis 3
    
    JDBC로 처리하는 부분의 코드와 파라미터 설정 및 결과 매핑을 대신 해주는 프레임워크
    
    [MyBatis - 마이바티스 3 | 소개](https://mybatis.org/mybatis-3/ko/index.html)
    
- MyBatis 시작하기
    1. resources 폴더 만들기 (new → Source Folder)
    2. resources/mybatis-config 파일 만들기(new → xml Files)
    - mybatis-config.xml
        
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
          <!-- 이 문서의 형식이 configuration(== 환경설정)임을 알려줌
          		=> configuration태그가 전체를 감싸고있음
          		DTD: 유효성을 체크해줌(태그들이 configuration 태그 안에 존재해도 되는지를 체크 -->
        <configuration>
        	<!-- settings : MyBatis 구동 시 선언할 설정들을 작성하는 영역 -->
        	<settings>
        		<!-- 만약에 null로 데이터가 전달되었다면 빈칸이 아닌 NULL로 인식하겠다.(무조건 NULL을 대문자로 작성!!) -->
        		<setting name="jdbcTypeForNull" value="NULL"/>
        	</settings>
        	
        	<!-- typeAlias : VO/DTO 클래스들의 풀 클래스명을 단순한 클래스명으로 사용하기 위해서 별칭을 등록할 수 있는 영역 -->
        	<typeAliases>
        	</typeAliases>
        	
        	<!-- environment : MyBatis에서 연동할 DB정보들을 등록할 영역(여러개의 DB정보를 등록 가능)
        		=> default속성으로 여러개의 id중 어떤 DB를 기본 DB로 사용할지 설정해줘야 함 -->
        	<environments default="develoment">
        		<environment id="develoment">
        			<!-- 
        				* transactionManager는 JDBC와 MANAGED 둘 중 하나를 선택할 수 있음
        				- JDBC : 트랜잭션을 내가 직접 관리하겠다. (수동 commit)
        				- MANAGED : 개발자가 트랜잭션에 대해서 어떠한 영향도 행사하지 않겠다. (자동 commit)
        			 -->
        			 <transactionManager type="JDBC"/>
        			 <!-- 
        			 	* dataSource는 POOLED랑 UNPOOLED 둘 중 하나를 선택할 수 있음(ConnectionPool 사용여부)
        			 	- ConnectionPool : Connectino 객체를 담아둘 수 있는 영역
        			 						한 번 생성된 Connection객체를 담아두면 재사용해서 쓸 수 있음
        			 	=> POOLED : 사용하겠다 / UNPOOLED : 안써~
        			  -->
        			 <dataSource type="POOLED">
        			 	<property name="driver" value="oracle.jdbc.driver.OracleDriver"/>
        			 	<property name="url" value="jdbc:oracle:thin:@localhost:1521:xe"/>
        			 	<property name="username" value="MYBATIS"/>
        			 	<property name="password" value="mybatis"/>
        			 </dataSource>
        		</environment>
        	</environments>
        	
        	<!-- mapper : 실행할 sql문들을 기록해둔 mapper파일들을 등록하는 영역 -->
        	<mappers>
        	</mappers>
        
        </configuration>
        ```
        
    

### MyBatis를 사용하여 DB 연동하기

- src/com/kh/mybatis/common/template/Template.java
    
    ```java
    package com.kh.mybatis.common.template;
    
    import java.io.IOException;
    import java.io.InputStream;
    
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    
    public class Template {
    	
    	// 마이바티스
    	public static SqlSession getSqlSession() {
    		// mybatis-config.xml파일을 읽어들여서
    		// 해당 DB와 접속된 SqlSession 객체를 생성해서 반환
    		SqlSession sqlSession = null;
    		
    		// SqlSession 객체를 생성하기 위해서는 SqlSessionFactory 객체 필요
    		// SqlSessionFactory 객체를 생성하기 위해서는 SqlSessionFactoryBuilder객체 필요
    		String resource = "/mybatis-config.xml";
    		// / 는 모든 소스폴더의 최상위 폴더들을 의미(resources, src)
    
    		try {
    			InputStream stream = Resources.getResourceAsStream(resource); // 자원으로부터 통로를 얻어내겠다.
    			
    			// 1단계 : new SqlSessionFactoryBuilder() : sqlSessionFactoryBuilder 객체 생성
    			// 2단계 : .build(stream) : 통로로부터 mybatis-config.xml파일을 읽어들여서 sqlSessionFactory객체를 만들겠다.
    			// 3단계 : .openSession(false) : sqlSession객체 생성 및 앞으로 트랜잭션 처리 시 자동으로 commit을 할건지 안할건지 여부 지정
    			// => false == 자동커밋을 하지 않겠다. == 개발자가 직접 commit하겠다
    			// => openSession() 기본값은 false이긴 함
    			sqlSession = new SqlSessionFactoryBuilder().build(stream).openSession(false);
    		} catch (IOException e) {
    			e.printStackTrace();
    		} 
    		
    		return sqlSession;
    	}
    	
    
    }
    ```
    
1. resources/mappers
    - mapper.xml
        
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
          <mapper namespace="메퍼이름">
          
          </mapper>
        ```
        
        - 작성법
            - namespace : 해당 mapper 파일의 고유한 별칭
            - DML문
                
                ```xml
                <insert id="각 sql문의 식별자" parameterType="전달받은 자바타입(풀클래스명) 혹은 별칭">
                  			SQL문
                </insert>
                <update></update>
                <delete></delete>
                ```
                
                ※ parameterType 속성은 전달받을 값이 없다면 생략 가능
                
                ※ 실행 결과가 처리된 행의 갯수(int)이기 때문에 resultType 또는 resultMap 작성할 필요 X
                
            - SELECT문
                
                ```xml
                <select id="각 sql문들의 식별자" parameterType="전달받을 자바타입(풀클래스명) 혹은 별칭"
                				resultType="조회결과를 반환하고자하는 자바타입" 또는 
                		  	resuleMap="조회결과를 뽑아서 매핑할 resultMap의 id">
                  	SQL문
                </select>
                ```
                
                ※ parameterType 속성은 전달받을 값이 없다면 생략 가능
                
                ※ 반드시 resultType(자바에서 제공하는 자료형) 또는 resultMap(내가 만든 VO클래스 타입)으로 결과값에 대한 타입 지정
                
            - ? 대신 sql문에 전달된 객체로부터 값을 꺼내서 #{필드명 또는 변수명 또는 map의 키값}
            
    1. mybatis-config.xml파일에 mapper 추가
        - mybatis-config.xml
            
            ```xml
            <!-- mapper : 실행할 sql문들을 기록해둔 mapper파일들을 등록하는 영역 -->
            	<mappers>
            			<mapper resource="/mappers/메퍼이름" /> <!-- 메퍼 경로 작성 -->
            	</mappers>
            ```
            
    2. mybatis-config.xml파일에 VO클래스 별칭 등록
        - mybatis-config.xml
            
            ```xml
            <!-- typeAlias : VO/DTO 클래스들의 풀 클래스명을 단순한 클래스명으로 사용하기 위해서 별칭을 등록할 수 있는 영역 -->
            <typeAliases>
            		<!-- type : vo경로 , alias : 별칭 -->
            		<typeAlias type="com.kh.mybatis.member.model.vo.Member" alias="member" /> 
            		<typeAlias type="com.kh.mybatis.board.model.vo.Board" alias="board" />
            		<typeAlias type="com.kh.mybatis.board.model.vo.Reply" alias="reply" />
            </typeAliases>
            ```
