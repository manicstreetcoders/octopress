---
layout: post
title: "ANTLR &amp; Language Implementation Patterns"
date: 2013-06-23 11:26
comments: true
categories: 
---

ANTLR 의 개발자 T.P. 가 Pragmatic 에서 책을 두 권 출판했다.

처음 예제는 List Parser 를 만드는 것.

리스트라 함은.

[ a,b,c] 나 [a, [x,y,z], b, c]

같은 것들.

### Token

Token 을 보관하는 데이터 구조.

Type 과 Text 로 이루어져있고, 

Type 으로는

    T_EOF
    T_NAME
    T_COMMA
    T_LBRACK
    T_RBRACK

등이 있다.

Text 에는 "<EOF>", "NAME", "COMMA", "LBRACK", "RBRACK" 등이.

Token class 의 method 로는,

new_token / print_token 정도.

### LL(1) Parser

기본적인 사상은,

lookahead 오퍼레이션과 consume 을 분리하는 것.

LL(1) Parser 에서는 말장난같지만, LL(k) Parser 로 가면 개념이 명료해짐.

기본적인 함수는 nextToken() - 버퍼를 tokenize 하는 함수.

whitespace 는 consume 해버린다.

',', '[', ']' 을 만나면, consume 하고, 새로운 토큰을 

    new_token(T_LBRACK, "[") 
    new_token(T_RBRACK, "]") 
    new_token(T_COMMA, ",") 

등으로 만들어 리턴.

Character buffer 를 tokenizing 해서 token stream 으로 변환.

### LL(k) Parser

LL(k) Parser 는 lookahead 를 몇 개 앞의 토큰까지 보고 판단하느냐가 핵심.

1 개의 lookahead 만 가지고 파싱을 진행할 수 있는 문법이 있는가하면,

불가능한 문법도 있기 때문에, LL(k) Parser 가 필요.

LL(1) 에서는 lookahead 토큰 하나만 가지고 했다면, LL(4) 에서는 4 개의 lookahead 토큰을 가지고 파싱 작업을 수행하게 된다.

LIST 를 그래머로 기술하면,

    list : '[' elements ']' ;
    elements : element (',' element)* ;
    element : NAME | list

    [ a, b, c ] 
    [ a, [ x, y, z ], b ] 

등을 기술할 수 있다.

위의 그래머는 다음 코드로 변환된다.

    list() { match(T_LBRACK); elements(); match(T_RBRACK); }
    elements() { element(); if (lookahead.type == T_COMMA) { match(T_COMMA); elements(); } }
    element() { 
      if (lookahead.type == T_NAME) match(T_NAME);
      else if (lookahead.type == T_LBRACK) list();
      else throw new Error("unexpeced: " + lookahead.type);
    }

결국 해보니, 위의 그래머는 lookahead 하나만 가지고 구현 가능하다. LL(1).

그럼 LL(2) 파서가 필요한 그래머를 만들어본다.

    list : '[' elements ']' ;
    elements : element (',' element)* ;
    element : NAME '=' NAME | NAME | list ;

`NAME = NAME` 과 `NAME` 이 동일한 `NAME` 으로 시작하기때문에, one lookahead 만으로는
파싱이 불가능하다. two lookaheads 를 유지하도록 한다.

Circular Lookahead Buffer 를 만들어 관리.

[a,b,c] 를 LL(4) 에서 예제.

    lookahead[0] '[' <-- bp
    lookahead[1] 'a'
    lookahead[2] ','
    lookahead[3] 'b'

가 초기 상태. 4 개의 lookahead 를 가지게 된다.

초기 lookahead 버퍼를 셋업하는 방법은, 

    lookahead[bp] = lexer.getToken();
    bp = (bp+1) % k;

를 k 번 만큼하면, `lookahead[0]` ~  `lookahead[k-1]` 까지의 버퍼가 토큰으로 채워지고, bp 는 0 으로 돌아와서, 처음을 가리키게 된다.
버퍼도 채우고, 맨 처음도 가리키고.

consume 을 한번 하면.

    lookahead[0] ','
    lookahead[1] 'a' <-- bp
    lookahead[2] ','
    lookahead[3] 'b'
    
첫 '[' 는 discard 되고, base pointer 가 전진했으며, 4 개의 lookahead 를 maintain 하기 위해, 새로운 ',' 가 들어왔다.

    lookahead = new Token[4];
    for (i=0;i<4;i++) lookahead[i] = lexer.nextToken();
    bp = 0;

하면 초기 CLB 셋업이 끝난다.

    consume() { lookahead[bp] = lexer.nextToken(); bp = (bp + 1) % 4; }

를 하면 자연스럽게 가장 오래된 것을 버리고, 새로운 토큰을 맨 뒤에 넣고, base pointer 는 circular buffer 의 처음을 가리키게 된다.

    LT(i) { return lookahead[bp + i - 1]; }

하면 1 번째, 2 번째 등등의 lookahead 를 할 수 있게 된다.

    "To visualize parsing decisions, imagine a maze with a single entrance and a
    single exit that has words written on the floor. Every sequence of words along
    a path from entrance to exit represents a sentence. The structure of the maze
    is analogous to the rules in a grammar that define a language. To test a sentence
    for membership in a language, we compare the sentence's words with
    the words along the floor as we traverse the maze. If we can get to the exit by
    following the sentence's words, that sentence is valid."

### JSON Grammar

예제삼아 JSON 을 ANTLR 그래머로 기술해본다.

    object
        : '{' pair (',' pair)* '}'
        | '{' '}'
        ;

    array
        : '[' value (',' value)* ']'
        | '[' ']'
        ;

    value
        : STRING
        | NUMBER
        | object
        | array
        | 'true'
        | 'false'
        | 'null'
        ;

### Building AST

ANTLR 를 사용하지 않고, 손으로 AST 를 만들어보았다.

AST 는 Homogeneous AST / Normalized Heterogeneous AST / Irregular Heterogeneous AST 정도로 
나눌 수 있는데, 간단한 Homogeneous AST 를 만들어 보았다.

Recursive 하게 decent 하는 파서의 특성을 활용하면, `AST.addChild(parent, int astType, Token token)` 
정도의 콜을 적절히 달아주는 것으로 트리 빌딩이 가능하다.

    public class AST {
        Token token;
        List<AST> children;
        public AST(Token token) { this.token = token; }
        public void addChild(AST t) { 
            if (children == null) children = new ArrayList<AST>();
            children.add(t);
        }
    }

일단 하나의 클래스로 모든 노드를 담는다.
하나의 타입만 있으니까, children 을 `List<AST> children` 으로 관리할 수 있다.
AST 를 만들기 위해, 3 lookaheads 를 사용하였다.

    $ java Test '[1,x=2,[3,4]]'
    match LBRACK
    match NAME
    match COMMA
    match NAME
    match EQUALS
    match NAME
    match COMMA
    match LBRACK
    match NAME
    match COMMA
    match NAME
    match RBRACK
    match RBRACK
    AST Type: AST_ROOT
        AST Type: AST_LIST
            AST Type: AST_NAME
                Token: NAME : 1
            AST Type: AST_ASSIGN
                AST Type: AST_IDENTIFIER
                    Token: NAME : x
                AST Type: AST_VALUE
                    Token: NAME : 2
            AST Type: AST_LIST
                AST Type: AST_NAME
                    Token: NAME : 3
                AST Type: AST_NAME
                    Token: NAME : 4

### Building AST with ANTLR

벡터 계산에 관한 문법을 만들고, ANTLR 를 가지고 AST 를 build 한다.

    x = 1+2
    y = 1*2+3
    z = [1,2] + [3,4]
    a = [1,2] . [3,4]
    b = 3 * [1,2]
    print x+2

    statline : stat+ ;
    stat: ID '=' expr | 'print' expr ;
    expr: multExpr ('+' multExpr)* ; // E.g., "3*4 + 9"
    multExpr: primary (('*'|'.') primary)* ;
    primary : INT | ID | '[' expr (',' expr)* ']'

우선 `output=AST;` 로 AST 를 빌드 모드로.

    grammar VecMathAST;
    options {output=AST;}
    tokens {VEC;}

그리고 기존 문법에 AST rewrite 를 위한 규칙을 추가한다. 예를 들어

    statlist : stat+ ;
    stat: ID '=' expr

같은 경우 `stat: ID '=' expr` 에 AST rewrite 규칙을 추가한다.

    statlist : stat+ ;
    stat: ID '=' expr -> ^('=' ID expr)

`^('=' ID expr)` 은, `'='` 가 subtree 의 root 고, `ID`, `expr` 가 children 으로,
AST 를 구성하라는 rewriting rule 이 된다.

Antlr 의 AST rewriting 을 이해하기 위해 참조한 블로그는 [Bart's Blog](http://bkiers.blogspot.kr/2011/03/5-building-ast.html) 임.

    bar : A B C D -> ^(B A C) ;

을 하면, B 가 root 가 되고, A, C 가 children 이 되고, D 는 없어진다.

다른 예.

    addExpr : multiplyExpr (('+' | '-') multiplyExpr)* ;

    addExpr : multiplyExpr (('+' | '-')^ multiplyExpr)* ;

을 하면, '+', '-' 가 root 가 되고 multiplyExpr 2 개가 children 이 된다.

root 로 할만한 적당한 녀석이 없을 경우에는 하나 만들면 된다. 

    block : (statement | funcitonDecl)* (Return expression ';')? ;

    block : 
        (statement | functionDecl)* (Return expression ';')?
            -> ^(BLOCK ^(STATEMENTS statement*) ^(RETURN expression?))
        ;

을 하면, 모든 block 은 STATEMENTS, RETURN 이라는 2 개의 children 을 가지게 된다.

본격적으로 AST rewriting rule 을 추가하면,

    // START: header
    grammar VecMathAST;
    options {output=AST;} // we want to create ASTs
    tokens {VEC;} // define imaginary token for vector literal
    // END: header

    // START: stat
    statlist : stat+ ;                    // builds list of stat trees
    stat: ID '=' expr  -> ^('=' ID expr)  // '=' is operator subtree root
        | 'print' expr -> ^('print' expr) // 'print' is subtree root
        ;
    // END: stat

    // START: expr
    expr:   multExpr ('+'^ multExpr)* ;        // '+' is root node
    multExpr: primary (('*'^|'.'^) primary)* ; // '*', '.' are roots
    primary
        :   INT   // automatically create AST node from INT's text
        |   ID    // automatically create AST node from ID's text
        |   '[' expr (',' expr)* ']' -> ^(VEC expr+)
        ;
    // END: expr

    ID  :   'a'..'z'+ ;
    INT :   '0'..'9'+ ;
    WS  :   (' '|'\r'|'\n')+ {skip();} ;

### Parse Tree

AST 와 Parse Tree 의 차이점이 [What's the difference between parse tree and AST?](http://stackoverflow.com/questions/5026517/whats-the-difference-between-parse-tree-and-ast) 에 정리되어 있다.

    The AST only contains all 'useful' elements that will be used for further processing,
    while the parse tree contains all the artifacts (spaces, brackets, ...)
    from the original document you parse

### Backtracking Parser & Memoizing Parser

LL(k) 파서로 판단하기 어려운 모호성이 존재할 때, 쓰는 기법.

    The easiest way to implement a backtracking strategy for a parsing
    decision is to speculatively attempt the alternatives in order until we
    find one that matches. Upon success, the parser rewinds the input and 
    parses the alternative normally. Upon failing to match an alternative,
    the parser rewinds the input and tries the next one. If the parser can't 
    find any matching alternative, it throws a "no viable alternative" 
    exception.

다음과 같은 문법을 보자. Python 과 같이 `[a,b] = [c,d]` 와 같은 리스트 어사인을 지원하는 문법이다.

    stat    : list EOF | assign EOF ;
    assign  : list '=' list ;
    list    : '[' elements ']' ;
    elements    : element (',' element)* ;
    element     : NAME '=' NAME | NAME | list

stat 의 경우, list 인지 assign 인지 정하기 위해서는 리스트를 파싱하고 나서, 토큰 하나를 더 봐야
알 수 있다. 

얼터너티브를 시도해보면서, 경우에 따라 백트래킹하는 코드.

    public void statement() throws RecognitionException {
        if ( speculate_stat_alt1() ) { list(); match(Lexer.EOF_TYPE); }
        else if ( speculate_stat_alt2() ) { assign(); match(Lexer.EOF_TYPE); }
        else throw new NoViableAltException("expecting stat found " + LT(1));
    }

    public boolean speculate_stat_al1() {
        boolean success = true;
        mark();
        try { list(); match(Lexer.EOF_TYPE); }
        catch (RecognitionException e) { success = false; }
        release();
        return success;
    }
    public boolean speculate_stat_al2() {
        boolean success = true;
        mark();
        try { assign(); match(Lexer.EOF_TYPE); }
        catch (RecognitionException e) { success = false; }
        release();
        return success;
    }

