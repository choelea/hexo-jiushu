---
title:  Java 正则表达式示例
description: Java 正则表达式示例
...



```
package test.com;

import static org.junit.Assert.*;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.junit.Test;
/**
 * Unit Test for domain regular expression
 * @author Joe
 */
public class DomainRegexTester {

	private final static Pattern mainDomainPattern = Pattern.compile("(m\\.)?((www|en|pt|es|ru)\\.)?okchem\\.com");
	
	private final static Pattern vipDomainPattern = Pattern.compile("(\\w{3,30})\\.(m\\.)?((en|pt|es|ru)\\.)okchem\\.com");
	
	public static boolean match(String regex, String text) {
		Pattern pattern = Pattern.compile(regex);
		Matcher matcher = pattern.matcher(text);
	    
	    return matcher.matches();
	}
	
	@Test
	public void testMatchGroup_NonVIP_PC() {		
		Matcher pcMatcherWww = mainDomainPattern.matcher("www.okchem.com");
	    if(pcMatcherWww.find()) {
	    	assertTrue("www".equals(pcMatcherWww.group(3)));
	    	assertTrue("www.".equals(pcMatcherWww.group(2)));
	    }
	    
	    Matcher pcMatcherEn = mainDomainPattern.matcher("en.okchem.com");
	    if(pcMatcherEn.find()) {
	    	assertTrue(pcMatcherEn.group(1)==null);
	    	assertTrue("en".equals(pcMatcherEn.group(3)));
	    }
	    
	    
	    Matcher mobileMatcherEn = mainDomainPattern.matcher("m.okchem.com");
	    if(mobileMatcherEn.find()) {
	    	assertTrue("m.".equals(mobileMatcherEn.group(1)));
	    } 
	    
	    Matcher mobileMatcherEs = mainDomainPattern.matcher("m.es.okchem.com");
	    if(mobileMatcherEs.find()) {
	    	assertTrue("es".equals(mobileMatcherEs.group(3)));
	    	assertTrue("es.".equals(mobileMatcherEs.group(2)));
	    }  
	   
	}
	
	
	@Test
	public void testMainDomainMatch() {		
	   assertTrue(matchMain("www.okchem.com"));
	   assertTrue(matchMain("pt.okchem.com"));
	   assertTrue(matchMain("es.okchem.com"));
	   assertTrue(matchMain("pt.okchem.com"));
	   assertTrue(matchMain("ru.okchem.com"));
	   
	   assertTrue(matchMain("m.okchem.com"));
	   assertTrue(matchMain("m.pt.okchem.com"));
	   assertTrue(matchMain("m.es.okchem.com"));
	   assertTrue(matchMain("m.ru.okchem.com"));	  
	   
	   assertTrue(matchMain("m.www.okchem.com"));
	   assertFalse(matchMain("m.www.jiu-shu.com"));
	}
	
	public void testVipDomainmatch() {
		/*** VIP store domain ***/
		assertTrue(matchVIP("vipcode.ru.okchem.com"));
		assertTrue(matchVIP("vipcode.en.okchem.com"));
		assertTrue(matchVIP("vipcode.es.okchem.com"));
		assertTrue(matchVIP("vipcode.pt.okchem.com"));

		assertTrue(matchVIP("vipcode.m.pt.okchem.com"));
		assertTrue(matchVIP("vipcode.m.ru.okchem.com"));
		assertTrue(matchVIP("vipcode.m.en.okchem.com"));
		assertTrue(matchVIP("vipcode.m.es.okchem.com"));

		assertFalse(matchVIP("v.pt.okchem.com"));// vipcode is too short
		assertTrue(matchVIP("abcdefghij12345678901234567980.en.okchem.com"));// vipcode is just 30
		assertFalse(matchVIP("abcdefghij123456789012345679801.pt.okchem.com"));// vipcode is too short

		assertFalse(matchVIP("v.m.pt.okchem.com"));// vipcode is too short
		assertTrue(matchVIP("abcdefghij12345678901234567980.m.pt.okchem.com"));// vipcode is just 30
		assertFalse(matchVIP("abcdefghij123456789012345679801.m.pt.okchem.com"));// vipcode is too short
	}
	
	public static boolean matchMain(String text) {
		Matcher matcher = mainDomainPattern.matcher(text);
	    return matcher.matches();
	}
	public static boolean matchVIP(String text) {
		Matcher matcher = vipDomainPattern.matcher(text);
	    return matcher.matches();
	}
	
}

```


```
package test.com;

import static org.junit.Assert.*;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.junit.Test;
/**
 * Unit Test for domain regular expression
 * @author Joe
 */
public class DomainRegexTester {

	private final static Pattern pattern = Pattern.compile("(\\w{3,30}\\.)?(m\\.)?((www|en|pt|es|ru)\\.)?okchem\\.com");
	
	
	public static boolean match(String regex, String text) {
		Pattern pattern = Pattern.compile(regex);
		Matcher matcher = pattern.matcher(text);
	    
	    return matcher.matches();
	}
	
	@Test
	public void testMatchGroup_NonVIP_PC() {		
		Matcher matcher1 = pattern.matcher("www.okchem.com");
	    if(matcher1.find()) {
	    	assertTrue("www.".equals(matcher1.group(1)));
	    	assertTrue(matcher1.group(2)==null);
	    	assertTrue(matcher1.group(3)==null);
	    	assertTrue(matcher1.group(4)==null);
	    }
	    
	    Matcher matcher2 = pattern.matcher("en.okchem.com");
	    if(matcher2.find()) {
	    	assertTrue("en".equals(matcher2.group(4)));
	    	assertTrue(matcher2.group(2)==null);
	    	assertTrue("en.".equals(matcher2.group(3)));
	    }
	}
	@Test
	public void testMatchGroup() {		
		Matcher matcher = pattern.matcher("www.okchem.com");
	    if(matcher.find()) {
	    	assertTrue("www.".equals(matcher.group(1)));
	    	assertTrue(matcher.group(2)==null);
	    	assertTrue(matcher.group(3)==null);
	    }
	}
	
	@Test
	public void testParseLanguage_NonLanguage() throws Exception {		
		assertTrue(parseLanguageFromText("www.okchem.com")==null);
		assertTrue(parseLanguageFromText("m.okchem.com")==null);
	}
	@Test
	public void testParseLanguage_HasLanguage() throws Exception {		
		assertTrue("pt".equals(parseLanguageFromText("pt.okchem.com")));
		assertTrue("pt".equals(parseLanguageFromText("m.pt.okchem.com")));
		assertTrue("ru".equals(parseLanguageFromText("vipcode.ru.okchem.com")));
		assertTrue("pt".equals(parseLanguageFromText("vipcode.m.pt.okchem.com")));
	}
		
	@Test
	public void testDomainMatch() {		
	   assertTrue(match("www.okchem.com"));
	   assertTrue(match("pt.okchem.com"));
	   assertTrue(match("es.okchem.com"));
	   assertTrue(match("pt.okchem.com"));
	   assertTrue(match("ru.okchem.com"));
	   
	   assertTrue(match("m.okchem.com"));
	   assertTrue(match("m.pt.okchem.com"));
	   assertTrue(match("m.es.okchem.com"));
	   assertTrue(match("m.ru.okchem.com"));
	   
	   /*** VIP store domain ***/
	   
	   assertTrue(match("vipcode.ru.okchem.com"));
	   assertTrue(match("vipcode.en.okchem.com"));
	   assertTrue(match("vipcode.es.okchem.com"));
	   assertTrue(match("vipcode.pt.okchem.com"));
	   
	   assertTrue(match("vipcode.m.pt.okchem.com"));
	   assertTrue(match("vipcode.m.ru.okchem.com"));
	   assertTrue(match("vipcode.m.en.okchem.com"));
	   assertTrue(match("vipcode.m.es.okchem.com"));   
	  
	   assertFalse(match("vi.pt.okchem.com"));// vipcode is too short	  
	   assertTrue(match("abcdefghij12345678901234567980.en.okchem.com"));// vipcode is just 30
	   assertFalse(match("abcdefghij123456789012345679801.pt.okchem.com"));// vipcode is too short
	   
	   assertFalse(match("vi.m.pt.okchem.com"));// vipcode is too short	   
	   assertTrue(match("abcdefghij12345678901234567980.m.pt.okchem.com"));// vipcode is just 30
	   assertFalse(match("abcdefghij123456789012345679801.m.pt.okchem.com"));// vipcode is too short
	   
	   assertTrue(match("m.www.okchem.com"));
	   assertFalse(match("m.www.jiu-shu.com"));
	}
	
	public static boolean match(String text) {
		Matcher matcher = pattern.matcher(text);
	    return matcher.matches();
	}
	
	public static String parseLanguageFromText(String text) throws Exception {
		Matcher matcherPc = pattern.matcher(text);
	    if(matcherPc.find()) {
	    	return matcherPc.group(4);
	    }
	    throw new Exception("Not Match");
	}
	
}

```