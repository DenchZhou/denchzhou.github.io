---
layout: post
title: Java使用POI解析Excel表格
categories: JavaSE
description: 使用POI解析Excel表格的demo
keywords: leetcode
---

### 概述
Excel表格是常用的数据存储工具，项目中经常会遇到导入Excel和导出Excel的功能。
常见的Excel格式有xls和xlsx。07版本以后主要以基于XML的压缩格式作为默认文件格式xlsx。新格式主要是使用了OpenXML标准，结合了XML与Zip压缩技术。在这里就不细说，感兴趣的读者可以自行去查找相关知识。本文将重点以这两种文件格式的解析来展开。
Excel主要有以下部分组成：
一个Excel相当于一个工作簿（WorkBook）;
每个sheet相当于一张表格；
sheet里面又由单元格Cell组成；
### 操作Excel的方式
Java提供了操作Excel的api JXL（Java Excel API），但是JXL只支持07版本以前，也就是xls后缀的Excel。因此使用中通常使用apache的POI

**maven中的POI依赖**
```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.0.1</version>
</dependency>
<!-- 07版本以后的格式 -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.0.1</version>
</dependency>
```
其中最顶层接口Workbook;主要有三个实现类XSSFWorkbook、HSSFWorkbook、SXSSFWorkbook。
![结构图.png](https://upload-images.jianshu.io/upload_images/14607771-e2e19183646204f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### XSSFWorkbook
XSSFWorkbook主要用于解析xlsx。
```
@Test
public void testXlsx(){
        File file = new File("C:\\Users\\Administrator\\Desktop\\pdf\\test.xlsx");
        if(!file.exists()){
            System.out.println("文件不存在");
            return ;
        }
        FileInputStream fis = null;
        Workbook workBook = null;
        try {
            fis = new FileInputStream(file);
            workBook = new XSSFWorkbook(fis); // 使用XSSFWorkbook
            dealWorkBook(workBook); // 将代码封装复用，见下一个方法

        } catch (Exception e) {
            e.printStackTrace();
        } finally{ //关流
            if(fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(workBook != null){
                try {
                    workBook.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
```
```
public void dealWorkBook(Workbook workBook){
        Sheet sheet = workBook.getSheetAt(0); // 获取第一个sheet
        Map<Integer, List<String>> map = new HashMap<Integer, List<String>>(); //第一个参数表示行数 第二个List保存该行的cell数据
        int i = 0;
        for(Row row : sheet){
            map.put(i, new ArrayList<String>());
            for(Cell cell : row){ // 遍历当前行的所有cell
                switch(cell.getCellType()) {
                    case STRING:
                        map.get(i).add(cell.getRichStringCellValue().getString()); // 如果是字符串则保存
                        break;
                    case _NONE:
                        break;
                    case NUMERIC:
                        map.get(i).add(cell.getNumericCellValue()+""); //将数值转换为字符串
                        break;
                    case BOOLEAN:
                        break;
                    case FORMULA:
                        break;
                    case BLANK:
                        break;
                    case ERROR:
                        break;
                }
            }
            i++;
        }
        Set<Integer> keys = map.keySet(); // 以下为遍历 Map看解析结果
        Iterator<Integer> it = keys.iterator();
        while(it.hasNext()){
            List<String> list = map.get(it.next());
            for(String s : list){
                System.out.print(s+"      ");
            }
            System.out.println();
        }
    }
```
运行结果如下
```
1.0      2.0      3.0      4.0      
a      b      c      d      
7.0      8.0      9.0      10.0      
e      f      g      h      
```
### HSSFWorkbook

HSSFWorkbook主要用于解析xls
代码如下
```
    @Test
    public void testXls() throws Exception{
        File file = new File("D:\\excelTest\\test2.xls");
        if(!file.exists()){
            System.out.println("文件不存在");
            return ;
        }
        FileInputStream fis = new FileInputStream(file);
        Workbook workBook = new HSSFWorkbook(fis, true); // 使用HSSFWorkbook 构造函数略有不同 true表示转化成为Nodes
        dealWorkBook(workBook); // 复用上面的方法
        workBook.close();
        fis.close();
    }
```
解析结果如下
```
1.0      2.0      3.0      4.0      
a      b      c      d      
7.0      8.0      9.0      10.0      
e      f      g      h      
11.0      12.0      13.0      14.0  
```
### 注
本文参考了https://blog.csdn.net/holmofy/article/details/82532311
