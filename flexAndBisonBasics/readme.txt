flex & bison basics

(for additional reference materials, see John R. Levine's O'Reilly book 'flex & bison' ISBN: 978-0-596-15597-1. This tutorial summarizes Chapter 1 in Levin's book.)

flex:

flex is an improved version of a library called lex, so we use the .l extension for all examples.

flex generates a scanner, which matches patterns and performs some programmer-specified behavior.

flex programs consist of three sections separated by %% lines.

Section 1 contains declarations and option setting. Code inside %{ and }% will appear at the start of the scanner's C source file.

Section 2 contains list of patterns and actions.
e.g. (from fb1-1.l)

%%

[a-zA-Z]+ { word++; chars += strlen(yytext); }
\n	  { chars++; lines++; }
.	  { chars++; }

%%

The pattern is given as a regular expression. When a pattern is matched, the action is executed. If an action block contains a return statement, then the scanner will return this value to the calling function and await a subsequent yylex() call to resume scanning after the last matched token.


Note: the variable yytext above contains a char* pointing to the string matched by the regex

Section 3 contains C code that is copied into the generated scanner program.
e.g. (from fb1-1.l)

%%

main(int arc, char **argv)
{
	yylex()
	printf("%8d%8d%8d\n", lines, words, chars);
}

The function yylex() calls the scanner. The scanner resumes after the last pattern matched, or at the beginning of the input if called for the first time.

You are encouraged to look through the scanner examples included in this directory.


bison:

bison allows us to parse the scanned input. bison is an improvement on yacc ("yet another compiler compilers"), so bison files use the .y extension.


Much like flex programs, bison programs contain three sections: declarations, rules, and C code.

In the declaration section, we include imports, variable declarations, and token declarations.

The rule section contains BNF rules for parsing the input and actions to execute when a rule is applied.

Here is an example rule form fb1-5.y:

factor: term
 | exp ADD factor { $$ = $1 + $3; }
 | exp SUB factor { $$ = $1 - $3; }
 ;

In the action expression above, $$ refers to the value of the token on the left of the BNF rule and $1 ... $N refer to the values of tokens 1 to N on the right side of the BNF rule.

The C code section contains code that bison will copy to the generated parser program. In the C code section, the yyparse() function calls the parser.

You can also define a function called yyerror(char *s) which is called when bison encounters an error while parsing the input.

See fb1-5.y for an example full bison program.


Compiling flex and bison together:

Here is an example makefile for fb1-5.y and fb1-5.l:

fb1-5: fb1-5.l fb1-5.y
       bison -d fb1-5.y
       flex fb1-5.l
       cc -o $@ fb1-5.tab.c lex.yy.c -lfl

bison -d calls runs bison (-d is the declaration flag) to produce the parser table fb1-5.tab.c and fb1-5.tab.h. flex fb1-5.l then calls the flex to create lex.yy.c. Then, we compile them together with the -lfl flag.

Compile and try out fb1-5 with some example expressions.

Note that the program correctly implemented multiplication/divison precedence over addition/subraction. How can you modify fb1-5.l and/or fb1-5.y to reverse this precedence?
