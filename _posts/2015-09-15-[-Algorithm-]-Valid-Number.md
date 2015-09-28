---
layout: post
category : Algorithm
tagline : "Valid Number"
tags : [Algorithm]
shortContent : "开心，英超“双红会“曼联3-1力克利物浦。新秀马夏尔真牛逼，戒骄戒躁，加油吧！"
---
{% include JB/setup %}

开心，英超“双红会“曼联3-1力克利物浦。新秀马夏尔真牛逼，戒骄戒躁，加油吧！

LeetCode上的算法题虽然测试用例不算太过苛刻，但也算是道道经典，还是能够把握住最基础和最关键的点。有些题也不乏有多种解法，把这些解法放在一起看，可为妙哉。这次分析一下LeetCode上通过率比较低的Valid Number。

题目是这样的：

	Validate if a given string is numeric.

	Some examples:
	"0" => true
	" 0.1 " => true
	"abc" => false
	"1 a" => false
	"2e10" => true
	
	Note: It is intended for the problem statement to be ambiguous. You should gather all requirements up front before implementing one.
	
	public class Solution {
		public boolean isNumber(String s) {
    	}
 	}

乍看这题，其实并不难，但之所以通过率这么低，是因为这道题需要考虑很多不同的输入情况。输入的字符串，可为整数，可为小数，可为正数，可为负数，可以是double类型，可以是float，也可以是科学计数法。因此，如何满足所有的这些条件，是这道题的关键。对于这道题，解法也有很多种。有从正面突击的，也有从后方巧入的。

我们列举一些合理的情况：

	1. 正负整数，例：2，-2；
	2. 正负小数，例：0.3，-1.2；
	3. 科学计数法，例：2e10，2E-8；

我们在列举一些需要注意的情况：
	
	1. 字符串有可能为空；
	2. 字符串有空格；
	3. 字符串不能有两个符号；
	4. 在java中1.4f和2.5D都是合法的，但在实际情况下，他不是个valid number；
	
#### 利用正则表达式

利用正则表达式可以凭借最短和最难懂的代码来解决问题，前提是你会正则表达式，或者。。。你能找到对的正则表达式。如下：

	public class Solution {
		public boolean isNumber(String s) {
			s = s.trim();
			if (s.length() == 0)
		 		return false;
		   	if (s.matches("[+-]?(([0-9]*\\.?[0-9]+)|([0-9]+\\.?[0-9]*))([eE][+-]?[0-9]+)?"))
		   		return true;
		   	else
	    		return false;
	    }
	}
	
当然，正则表达式也不是唯一的，相同的规则能写出很多种表达式。如下：

	(\\s*)[+-]?((\\.[0-9]+)|([0-9]+(\\.[0-9]*)?))(e[+-]?[0-9]+)?(\\s*)
or

	(-|\\+)?(([0-9]+(e(-|\\+)?[0-9]+)?)|([0-9]*\\.[0-9]+(e?(-|\\+)?[0-9]+)?)|([0-9]+\\.[0-9]*(e?(-|\\+)?[0-9]+)?))$

等等。。。


#### 利用Exception

利用Java已封装的一些类来处理这道题是个取巧的做法，那么，等于将这个输入的字符串交给Java来判断，如果是数，返回true，不是的话，就抛出异常，也就是返回false。如下：

	public class Solution {
		public boolean isNumber(String s) {
			s = s.trim();
			if (s.endsWith("f") ||s.endsWith("F") || s.endsWith("d")|| s.endsWith("D"))
				return false;
			try {
				Double.parseDouble(s);
			} catch (Exception e) {
				return false;
			}
			return true;
		}
	}


#### 老老实实考虑各种情况

其实出题者的意图并不是让我们弄个正则表达式一寥寥之，也不是希望我们用语言提供的工具类或者方法来绕开障碍，而是希望我们能有严谨的思维来解决问题。当我们把所有的情况都列出来之后，其实问题也并不复杂。

	public class Solution {
	    public boolean isNumber(String s) {
	    	//如果字符串s为null
	        if (s == null) return false;
			//去除字符串s两端的空格
	        s = s.trim();
	        
	        int n = s.length();
	        //如果字符串s的长度为0
	        if (n == 0) return false;

	        //一系列标志位
	        //符号数
	        int signCount = 0;
	        //是否含有eE
	        boolean hasE = false;
	        //是否有数字
	        boolean hasNum = false;
	        //是否有小数点
	        boolean hasPoint = false;
			//遍历字符串
	        for (int i = 0; i < n; i++) {
	            char c = s.charAt(i);

	            // 为不合法的字符
	            if (!isValid(c)) return false;

	            // 如果字符是数字，将有数字置为true
	            if (c >= '0' && c <= '9') hasNum = true;

	            // 如果字符是e或者E
	            if (c == 'e' || c == 'E') {
	                // 如果已经有E或者e,或者在e的前面没有数字
	                if (hasE || !hasNum) return false;
	                // e或者E不能在字符串的末尾
	                if (i == n - 1) return false;
	                //字符串有e或者E了
	                hasE = true;
	            }

	            // 如果字符是小数点
	            if (c == '.') {
	                // 小数点不能出现两次，也不能出现在e或者E的后面
	                if (hasPoint || hasE) return false;
	                // 如果小数点在最后一位，那么他前面必须是数字
	                if (i == n - 1 && !hasNum) return false;
					//字符串有小数点了
	                hasPoint = true;
	            }

	            // 如果字符是符号
	            if (c == '+' || c == '-') {
	                // 符号不能多于2个
	                if (signCount == 2) return false;
	                // 符号不能在末尾
	                if (i == n - 1) return false;
	                // 如果符号不再开头，那么他前面肯定是e或者E
	                if (i > 0 && !hasE) return false;
					//符号数加1
	                signCount++;
	            }
	        }

	        return true;
	    }
		//用于判断是否是合法的字符，包括小数点，加减号，e和E还有0-9数字
	    boolean isValid(char c) {
	        return c == '.' || c == '+' || c == '-' || c == 'e' || c == 'E' || c >= '0' && c <= '9';
	    }
	}

怎么样，其实也并不是很复杂，注意和留心各中合理和不合理的情况即可。

#### 面向对象的解法

当然，还有更加有追求的解法，既然是Java，作为一个面向对象的语言，那只写一个函数是不能满足我们的要求的，我们要追求其扩展性。

	interface NumberValidate {
    	boolean validate(String s);
	}
	
	abstract class  NumberValidateTemplate implements NumberValidate{
		public boolean validate(String s){
        	if (checkStringEmpty(s)){
            	return false;
        	}
        	s = checkAndProcessHeader(s);
        	if (s.length() == 0){
            	return false;
        	}
        	return doValidate(s);
    	}

    	private boolean checkStringEmpty(String s){
        	if (s.equals("")){
            	return true;
        	}
        	return false;
    	}

    	private String checkAndProcessHeader(String value){
        	value = value.trim();
        	if (value.startsWith("+") || value.startsWith("-")){
            	value = value.substring(1);
        	}
        	return value;
    	}
    	protected abstract boolean doValidate(String s);
	}

	class NumberValidator implements NumberValidate {

    	private ArrayList<NumberValidate> validators = new ArrayList<NumberValidate>();

    	public NumberValidator(){
        	addValidators();
    	}

    	private  void addValidators(){
        	NumberValidate nv = new IntegerValidate();
        	validators.add(nv);

        	nv = new FloatValidate();
        	validators.add(nv);

        	nv = new HexValidate();
        	validators.add(nv);
	
        	nv = new SienceFormatValidate();
        	validators.add(nv);
    	}

    	@Override
    	public boolean validate(String s){
        	for (NumberValidate nv : validators){
            	if (nv.validate(s) == true){
                	return true;
            	}
        	}
        	return false;
    	}
	}

	class IntegerValidate extends NumberValidateTemplate{

    	protected boolean doValidate(String integer){
        	for (int i = 0; i < integer.length(); i++){
            	if(Character.isDigit(integer.charAt(i)) == false) {
                	return false;
            	}
        	}
        	return true;
    	}
	}

	class HexValidate extends NumberValidateTemplate{

    	private char[] valids = new char[] {'a', 'b', 'c', 'd', 'e', 'f'};
    	protected boolean doValidate(String hex){
        hex = hex.toLowerCase();
        	if (hex.startsWith("0x")){
            	hex = hex.substring(2);
        	}else{
            	return false;
        	}

        	for (int i = 0; i < hex.length(); i++){
            	if (Character.isDigit(hex.charAt(i)) != true && isValidChar(hex.charAt(i)) != true){
                	return false;
            	}
        	}

        	return true;
    	}

    	private boolean isValidChar(char c){
        	for (int i = 0; i < valids.length; i++){
            	if (c == valids[i]){
                	return true;
            	}
        	}
        	return false;
    	}
	}

	class SienceFormatValidate extends NumberValidateTemplate{
		protected boolean doValidate(String s){
        	s = s.toLowerCase();
        	int pos = s.indexOf("e");
        	if (pos == -1){
            	return false;
        	}

        	if (s.length() == 1){
            	return false;
        	}

        	String first = s.substring(0, pos);
        	String second = s.substring(pos+1, s.length());

        	if (validatePartBeforeE(first) == false || validatePartAfterE(second) == false){
            	return false;
        	}
        	return true;
    	}

    	private boolean validatePartBeforeE(String first){
        	if (first.equals("") == true){
            	return false;
        	}

        	if (checkHeadAndEndForSpace(first) == false){
            	return false;
        	}

        	NumberValidate integerValidate = new IntegerValidate();
        	NumberValidate floatValidate = new FloatValidate();
        	if (integerValidate.validate(first) == false && floatValidate.validate(first) == false){
            	return false;
        	}
        	return true;
    	}

		private boolean checkHeadAndEndForSpace(String part){
        	if (part.startsWith(" ") ||
                part.endsWith(" ")){
            	return false;
        	}
	        return true;
    	}

    	private boolean validatePartAfterE(String second){
        	if (second.equals("") == true){
            	return false;
        	}

        	if (checkHeadAndEndForSpace(second) == false){
            	return false;
        	}

        	NumberValidate integerValidate = new IntegerValidate();
        	if (integerValidate.validate(second) == false){
            	return false;
        	}
        	return true;
    	}
	}

	class FloatValidate extends NumberValidateTemplate{
   		protected boolean doValidate(String floatVal){
        	int pos = floatVal.indexOf(".");
        	if (pos == -1){
            	return false;
        	}

        	if (floatVal.length() == 1){
            	return false;
        	}

        	NumberValidate nv = new IntegerValidate();
        	String first = floatVal.substring(0, pos);
        	String second = floatVal.substring(pos + 1, floatVal.length());

        	if (checkFirstPart(first) == true && checkFirstPart(second) == true){
            	return true;
        	}
        	return false;
    	}

    	private boolean checkFirstPart(String first){
        	if (first.equals("") == false && checkPart(first) == false){
            	return false;
        	}
        	return true;
    	}

    	private boolean checkPart(String part){
       		if (Character.isDigit(part.charAt(0)) == false ||
                Character.isDigit(part.charAt(part.length() - 1)) == false){
            	return false;
        	}

        	NumberValidate nv = new IntegerValidate();
        	if (nv.validate(part) == false){
            	return false;
        	}
        	return true;
    	}
	}

	public class Solution {
    	public boolean isNumber(String s) {
        	NumberValidate nv = new NumberValidator();
			return nv.validate(s);
    	}
	}

如果有新的条件加入，只需加入一个新的NewValidate继承NumberValidateTemplate，在doValidate中实现自己的验证逻辑即可。

#### 总结
有时候，看似简单的一道算法题，可以从各种角度切入，能引出各种奇妙的解法。
