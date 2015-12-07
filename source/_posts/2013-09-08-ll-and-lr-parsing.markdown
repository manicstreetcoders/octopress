---
layout: post
title: "LL and LR parsing"
date: 2013-09-08 20:17
comments: true
categories: 
---

[LL and LR parsing demystified](http://blog.reverberate.org/2013/07/ll-and-lr-parsing-demystified.html) 을 보고.

저자의 point 는 LL 과 LR 의 차이는 traversal 이라는 것.

파서는 traversal 이라는 인사이트를 제시한다.

blog 에 나온 postfix calculator

    ops = {
        '+': (lambda a,b: a+b),
        '-': (lambda a,b: a-b),
        '*': (lambda a,b: a*b)
    }

    def eval(tokens):
        stack = []

        for tk in tokens:
            if (tk in ops):
                a2 = stack.pop()
                a1 = stack.pop()
                stack.append(ops[tk](a1,a2))
            else:
                stack.append(int(tk))
        return stack.pop()

    if __name__ == "__main__":
        print eval("7 2 3 * +".split())

blog 에서 말한 infix -> postfix 변환하기.

tree building 없이 stack 만 가지고 Prefix 에서 Postfix 로 변환해본다.

    ops = { 
        "+": "plus", 
        "-": "minus",
        "*": "mutiply"
        }

    class Visit:
        tokens = []
        index = 0
        stack = []
        rev_polish = []

        def __init__(self, expr):
            self.tokens = expr.split()
            self.index = 0

        def visit(self):
            if (self.tokens[self.index] in ops):
                self.stack.append(self.tokens[self.index])
                self.index += 1
                self.visit() 
                self.visit()
                self.rev_polish.append(self.stack.pop())
            else:
                self.rev_polish.append(self.tokens[self.index])
                self.index += 1

    if __name__ == "__main__":
        visit = Visit("* + 1 2 3")
        visit.visit()
        print visit.rev_polish

    code% python polish.py
    ['1', '2', '+', '3', '*']

그 다음에 Infix parsing 트리를 만들어봐야겠다.

일단 문법은 NUMBER + NUMBER 형태만 지원.

    class Lexer:
        token = []
        index = 0

        def __init__(self, exp):
            self.token = exp.split()
            self.lookahead = self.nextToken()

        def nextToken(self):
            if self.index < len(self.token):
                tk = self.token[self.index]
                self.index += 1
                return tk
            else:
                return ""
        def consume(self):
            self.lookahead = self.nextToken()

    class Parser:
        def __init__(self, lexer):
            self.lexer = lexer
        def expr(self):
            self.match_number()
            self.match_op()
            self.match_number()
        def match_number(self):
            print "NUMBER %d" % (int(lexer.lookahead))
            self.lexer.consume()
        def match_op(self):
            print "OP %s" % (lexer.lookahead)
            self.lexer.consume()

    if __name__ == "__main__":
        lexer = Lexer("1 + 2")
        parser = Parser(lexer)
        parser.expr()

    blog% python expr.py
    NUMBER 1
    OP +
    NUMBER 2
