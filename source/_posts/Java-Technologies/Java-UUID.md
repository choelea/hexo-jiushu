---
title: Java 短ID 随机字符串
description: Java 短ID 随机字符串
---
使用apache common的lib 包。 用1000,0000 次测试，无重复字符串。

```
import java.io.File;
import java.io.IOException;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import org.apache.commons.io.FileUtils;
import org.apache.commons.lang.RandomStringUtils;
/**
 * 短ID 测试，使用1000,0000 次测试无重复字符串。 main1()生成500,0000。 后再用main2() 读取，然后再次新加500,0000生成，无重复记录。 
 * @author Joe
 *
 */
public class RandomStringUtilsTrial {
	public static void main(String[] args) throws IOException {	    
		main2();
	  }
  public static void main1() throws IOException {
    System.out.print("8 char string  >>>");
    Set<String> set = new HashSet<String>();
    for (int i = 0; i < 5000000; i++) {
    	set.add(RandomStringUtils.random(9, true, true));
	}  
    FileUtils.writeLines(new File("test.txt"), set);	
  }
  
  public static void main2() throws IOException {
	    System.out.print("8 char string  >>>");
	    Set<String> set = new HashSet<String>();
	    List<String> list = FileUtils.readLines(new File("test.txt"));
	    for (String string : list) {
	    	set.add(string);
		}
	    for (int i = 0; i < 5000000; i++) {
	    	set.add(RandomStringUtils.random(9, true, true));
		} 
	    System.out.println(set.size());
		
	  }
}
```