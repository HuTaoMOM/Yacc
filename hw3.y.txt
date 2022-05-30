%{
#include <stdio.h>
int yylex();
void yyerror(const char* s);
/*함수, 연산자, int, char, 포인터, 배열, 선택문, 반복문, return의 수를 셀 수 있도록
각각 해당하는 변수들을 선언하였다.*/
int function_count, operator_count, int_count, char_count =0;
int pointer_count, array_count, selection_count, loop_count, return_count = 0;
/* 타입을 체크해 int와 char의 경우를 알 수 있도록 checktype변수를 두었다.  
한개의 타입 뒤에 여러개의 변수가 오는 int a,b,c =0; 과 같은 경우에 각각을
세기 위해 nint와 nchar 변수를 두었다.*/
int checktype = 0; int nint, nchar =0;
%}


%token IDENTIFIER CONSTANT STRING_LITERAL SIZEOF
%token PTR_OP INC_OP DEC_OP LEFT_OP RIGHT_OP LE_OP GE_OP EQ_OP NE_OP
%token AND_OP OR_OP MUL_ASSIGN DIV_ASSIGN MOD_ASSIGN ADD_ASSIGN
%token SUB_ASSIGN LEFT_ASSIGN RIGHT_ASSIGN AND_ASSIGN
%token XOR_ASSIGN OR_ASSIGN TYPE_NAME
%token TYPEDEF EXTERN STATIC AUTO REGISTER
%token CHAR SHORT INT LONG SIGNED UNSIGNED FLOAT DOUBLE CONST VOLATILE VOID
%token STRUCT UNION ENUM ELLIPSIS
%token CASE DEFAULT IF ELSE SWITCH WHILE DO FOR GOTO CONTINUE BREAK RETURN

%start translation_unit


%%
primary_expression
: IDENTIFIER
| CONSTANT
| STRING_LITERAL
| '(' expression ')'
;
postfix_expression
: primary_expression
| postfix_expression '[' expression ']'
| postfix_expression '(' ')' 				{function_count++;}
//매개변수가 없는 함수가 선언되면, function_count의 개수를 증가시킨다.
| postfix_expression '(' argument_expression_list ')'	{function_count++;}
//매개변수가 있는 함수가 선언되면, function_count의 개수를 증가시킨다.
| postfix_expression '.' IDENTIFIER
| postfix_expression PTR_OP IDENTIFIER		{operator_count++;}
// "->" 연산자가 사용되면, operator_count의 개수를 증가시킨다.
| postfix_expression INC_OP				{operator_count++;}
//"++"가 후위연산자로 사용되면, operator_count의 개수를 증가시킨다.
| postfix_expression DEC_OP				{operator_count++;}
//"--"가 후위연산자로 사용되면, operator_count의 개수를 증가시킨다.
;
argument_expression_list
: assignment_expression
| argument_expression_list ',' assignment_expression
;
unary_expression
: postfix_expression
| INC_OP unary_expression				{operator_count++;}
//"++"가 전위연산자로 사용되면, operator_count의 개수를 증가시킨다.
| DEC_OP unary_expression				{operator_count++;}
//"--"가 전위연산자로 사용되면, operator_count의 개수를 증가시킨다.
| unary_operator cast_expression
| SIZEOF unary_expression
| SIZEOF '(' type_name ')'
;
unary_operator
: '&'
| '*'
| '+'
| '-'
| '~'
| '!'
;
cast_expression
: unary_expression
| '(' type_name ')' cast_expression				{operator_count++;}
//형 변환 연산자 사용 시 operator_count의 개수를 증가시킨다.
;
multiplicative_expression
: cast_expression
// "*", "/", "%" 산술 연산자이므로 operator_count의 개수를 증가시킨다.
| multiplicative_expression '*' cast_expression			{operator_count++;}
| multiplicative_expression '/' cast_expression			{operator_count++;}
| multiplicative_expression '%' cast_expression			{operator_count++;}
;
additive_expression
: multiplicative_expression
// "+", "-" 산술 연산자 이므로 operator_count의 개수를 증가시킨다.
| additive_expression '+' multiplicative_expression		{operator_count++;}
| additive_expression '-' multiplicative_expression		{operator_count++;}
;
shift_expression
: additive_expression
// "<<", >>" 쉬프트 연산자 이므로 operator_count의 개수를 증가시킨다.
| shift_expression LEFT_OP additive_expression			{operator_count++;}
| shift_expression RIGHT_OP additive_expression			{operator_count++;}
;
relational_expression
: shift_expression
/* "<=", ">=", "<", >" 비교연산자 이므로 operator_count의 개수를 증가시킨다.*/
| relational_expression '<' shift_expression			{operator_count++;}
| relational_expression '>' shift_expression			{operator_count++;}
| relational_expression LE_OP shift_expression			{operator_count++;}
| relational_expression GE_OP shift_expression			{operator_count++;}
;
equality_expression
: relational_expression
/*"==", "!=" 비교연산자 이므로 operator_count의 개수를 증가시킨다.*/
| equality_expression EQ_OP relational_expression		{operator_count++;}
| equality_expression NE_OP relational_expression		{operator_count++;}
;
and_expression
: equality_expression
//"&" 논리연산자 이므로 operator_count의 개수를 증가시킨다.
| and_expression '&' equality_expression			{operator_count++;}
;
exclusive_or_expression
: and_expression
//"^" 논리연산자 이므로 operator_count의 개수를 증가시킨다.
| exclusive_or_expression '^' and_expression			{operator_count++;}
;
inclusive_or_expression				
: exclusive_or_expression
//"|" 논리연산자 이므로 operator_count의 개수를 증가시킨다.
| inclusive_or_expression '|' exclusive_or_expression		{operator_count++;}
;
logical_and_expression				
: inclusive_or_expression
//"&&" 논리연산자 이므로 operator_count의 개수를 증가시킨다.
| logical_and_expression AND_OP inclusive_or_expression		{operator_count++;}
;
logical_or_expression
: logical_and_expression
//"||" 논리연산자 이므로 operator_count의 개수를 증가시킨다.
| logical_or_expression OR_OP logical_and_expression		{operator_count++;}
;
conditional_expression
: logical_or_expression
| logical_or_expression '?' expression ':' conditional_expression
;
assignment_expression
: conditional_expression
//"+=", "-=","*=","/=" 등의 대입 연산자들이므로 operator_count의 개수를 증가시킨다.
| unary_expression assignment_operator assignment_expression	{operator_count++;}
;
assignment_operator
: '='
| MUL_ASSIGN
| DIV_ASSIGN
| MOD_ASSIGN
| ADD_ASSIGN
| SUB_ASSIGN
| LEFT_ASSIGN
| RIGHT_ASSIGN
| AND_ASSIGN
| XOR_ASSIGN
| OR_ASSIGN
;
expression
: assignment_expression
| expression ',' assignment_expression
;
constant_expression
: conditional_expression
;
declaration
/*type_specifier에서 checktype을 변수의 자료형 타입이 int면 2, char면 1로 확인한다. 
init_declarator_list 부분에서 nint와 nchar변수를 통해 각 자료형의 변수가 몇개 선언되었는지 확인하고,
declaration부분에서 그 개수만큼 더하도록 했다.*/
: declaration_specifiers ';'					
{
	if(checktype ==1) char_count= char_count + nchar;
	else if(checktype==2) int_count = int_count + nint;
}
| declaration_specifiers init_declarator_list ';'			
{
	if(checktype ==1) char_count= char_count + nchar;
	else if(checktype==2) int_count = int_count + nint;
}
;
declaration_specifiers
: storage_class_specifier
| storage_class_specifier declaration_specifiers
| type_specifier
| type_specifier declaration_specifiers
| type_qualifier
| type_qualifier declaration_specifiers
;
init_declarator_list
/*type_specifier 에서 변수의 자료형을 checktype을 통해 설정하고, 
그 값(1이면 char, 2이면 int)에 따라 1또는 2라면 해당되는 자료형의 변수가 몇개 선언 되었는지
nint 와 nchar을 통해 세줬다. */
: init_declarator
{
	if(checktype ==1) nchar++;
	else if(checktype==2) nint++;
}
| init_declarator_list ',' init_declarator
{
	if(checktype ==1) nchar++;
	else if(checktype==2) nint++;
}
;
init_declarator
: declarator
//"="연산자 이므로 operator_count의 개수를 증가시킨다.
| declarator '=' initializer		{operator_count++;}
;
storage_class_specifier
: TYPEDEF
| EXTERN
| STATIC
| AUTO
| REGISTER
;
type_specifier
/*자료형을 읽어들일 때, int와 char을 제외한 나머지 타입은 checktype을 0으로 설정하고
char일 경우 1, int일 경우 2로 설정했다. 이후 init_declarator_list에서 자료형을 셀 때,
타입 토큰을 하나 들여올 때 마다 0으로 설정해서 중복해서 더해지는 일이 없도록 했다.*/
: VOID				{checktype=0;}
| CHAR				{checktype=1; nchar =0;}
| SHORT				{checktype=0;}
| INT				{checktype=2; nint =0;}
| LONG				{checktype=0;}
| FLOAT				{checktype=0;}
| DOUBLE				{checktype=0;}
| SIGNED				{checktype=0;}
| UNSIGNED			{checktype=0;}
| struct_or_union_specifier		{checktype=0;}
| enum_specifier			{checktype=0;}
| TYPE_NAME			{checktype=0;}
;
struct_or_union_specifier
: struct_or_union IDENTIFIER '{' struct_declaration_list '}'
| struct_or_union '{' struct_declaration_list '}'
| struct_or_union IDENTIFIER
;
struct_or_union
: STRUCT
| UNION
;
struct_declaration_list
: struct_declaration
| struct_declaration_list struct_declaration
;
struct_declaration
: specifier_qualifier_list struct_declarator_list ';'
;
specifier_qualifier_list
: type_specifier specifier_qualifier_list
| type_specifier
| type_qualifier specifier_qualifier_list
| type_qualifier
;
struct_declarator_list
: struct_declarator
| struct_declarator_list ',' struct_declarator
;
struct_declarator
: declarator
| ':' constant_expression
| declarator ':' constant_expression
;
enum_specifier
: ENUM '{' enumerator_list '}'
| ENUM IDENTIFIER '{' enumerator_list '}'
| ENUM IDENTIFIER
;
enumerator_list
: enumerator
| enumerator_list ',' enumerator
;
enumerator
: IDENTIFIER
| IDENTIFIER '=' constant_expression
;
type_qualifier
: CONST
| VOLATILE
;
declarator
//포인터가 나오는 곳이므로 pointer_count의 개수를 증가시킨다.
: pointer direct_declarator	{pointer_count++;}
| direct_declarator
;
direct_declarator
: IDENTIFIER
| '(' declarator ')'
//배열이 나오는 곳이므로 array_count의 개수를 증가시킨다.
| direct_declarator '[' constant_expression ']'	{array_count++;}
//배열이 나오는 곳이므로 array_count의 개수를 증가시킨다.
| direct_declarator '[' ']'			{array_count++;}
| direct_declarator '(' parameter_type_list ')'
| direct_declarator '(' identifier_list ')'
| direct_declarator '(' ')'
;
pointer
: '*'
| '*' type_qualifier_list
| '*' pointer
| '*' type_qualifier_list pointer
;
type_qualifier_list
: type_qualifier
| type_qualifier_list type_qualifier
;
parameter_type_list
: parameter_list
| parameter_list ',' ELLIPSIS
;
parameter_list
: parameter_declaration
| parameter_list ',' parameter_declaration
;
parameter_declaration
/*매개변수의 자료형이 int 또는 char인 경우를 확인하여,
int 이면 int_counter, char이면 char_counter의 개수를 증가시킨다.*/
: declaration_specifiers declarator
{
	if(checktype ==1) {char_count++;checktype=0;}
	else if(checktype==2) {int_count++;checktype=0;}
}
| declaration_specifiers abstract_declarator
| declaration_specifiers
;
identifier_list
: IDENTIFIER
| identifier_list ',' IDENTIFIER
;
type_name
: specifier_qualifier_list
| specifier_qualifier_list abstract_declarator
;
abstract_declarator
: pointer
| direct_abstract_declarator
| pointer direct_abstract_declarator
;
direct_abstract_declarator
: '(' abstract_declarator ')'
| '[' ']'
| '[' constant_expression ']'
| direct_abstract_declarator '[' ']'
| direct_abstract_declarator '[' constant_expression ']'
| '(' ')'
| '(' parameter_type_list ')'
| direct_abstract_declarator '(' ')'
| direct_abstract_declarator '(' parameter_type_list ')'
;
initializer
: assignment_expression
| '{' initializer_list '}'
| '{' initializer_list ',' '}'
;
initializer_list
: initializer
| initializer_list ',' initializer
;
statement
: labeled_statement
| compound_statement
| expression_statement
| selection_statement
| iteration_statement
| jump_statement
;
labeled_statement
: IDENTIFIER ':' statement
| CASE constant_expression ':' statement
| DEFAULT ':' statement
;
compound_statement
: '{' '}'
| '{' statement_list '}'
| '{' declaration_list '}'
| '{' declaration_list statement_list '}'
;
declaration_list
: declaration
| declaration_list declaration
;
statement_list
: statement
| statement_list statement
;
expression_statement
: ';'
| expression ';'
;
selection_statement
//선택문이 나오면 selection_count의 개수를 증가시킨다.
: IF '(' expression ')' statement 		{selection_count++;}
| SWITCH '(' expression ')' statement		{selection_count++;}
;
iteration_statement
//반복문이 나오는 경우 loop_count의 개수를 증가시킨다. 
: WHILE '(' expression ')' statement					{loop_count++;}
| DO statement WHILE '(' expression ')' ';'				{loop_count++;}
| FOR '(' expression_statement expression_statement ')' statement		{loop_count++;}
| FOR '(' expression_statement expression_statement expression ')' statement {loop_count++;}
/*강의록에 있는 ANSI C Yacc grammar 에서의 iteration_statement 부분의 FOR 의 규칙은
"for(int i = 0 ; i< n ; i++){}"과 같이 초기식에서 선언을 한 경우를 인식하지 못하여서,
밑에 두가지 경우를 추가했다.*/
| FOR '(' declaration expression_statement ')' statement			{loop_count++; }
| FOR '(' declaration expression_statement expression ')' statement		{loop_count++; }
;
jump_statement
: GOTO IDENTIFIER ';'
| CONTINUE ';'
| BREAK ';'
//"return"이 나오는 부분이다. return_count의 개수를 증가시킨다.
| RETURN ';'		{return_count++;}
| RETURN expression ';'	{return_count++;}
;
translation_unit
: external_declaration
| translation_unit external_declaration
;
external_declaration
: function_definition
| declaration
;
function_definition
//함수가 될 수 있는 경우를 확인하는 부분으로, function_count의 개수를 증가시킨다.
: declaration_specifiers declarator declaration_list compound_statement	{function_count++;}
| declaration_specifiers declarator compound_statement			{function_count++;}
| declarator declaration_list compound_statement			{function_count++;}
| declarator compound_statement					{function_count++;}
;
%%

int main(){
/*yyparse()함수를 실행시켜 yylex()를 통해 토큰을 받아들이고 주어진 문법에 맞게 분석한다.
yyparse() 함수의 실행이 끝난 후 변수의 값을 출력하도록 하였다.*/
	yyparse();
	printf("function = %d\n",function_count);
	printf("operator = %d\n",operator_count);
	printf("int = %d\n",int_count);
	printf("char = %d\n",char_count);
	printf("pointer = %d\n",pointer_count);
	printf("array = %d\n",array_count);
	printf("selection = %d\n",selection_count);
	printf("loop = %d\n",loop_count);
	printf("return = %d\n",return_count);
	return 0;
}

void yyerror(const char *str){
	fprintf(stderr, "error: %s\n",str);
}