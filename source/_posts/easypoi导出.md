---
title: easypoi导出
date: 2021-06-18 11:26:02
tags:
---

### 导出word

先导入依赖

```xml
<dependency>
    <groupId>cn.afterturn</groupId>
    <artifactId>easypoi-spring-boot-starter</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>ooxml-schemas</artifactId>
    <version>1.1</version>
</dependency>
```

```java
/**
 * 涉密岗位登记导出docx
 *  @param request
 * @param vxSecretPersonApply
 * @return
 */
@RequestMapping(value = "/downloadSecretRank")
public ModelAndView downloadSecretRank(HttpServletRequest request, VxSecretPersonApply vxSecretPersonApply) throws Exception {
 String property = System.getProperty("user.dir");
 // Step.1 组装查询条件查询数据
 QueryWrapper<VxSecretPersonApply> queryWrapper = QueryGenerator.initQueryWrapper(vxSecretPersonApply, request.getParameterMap());
 LoginUser sysUser = (LoginUser) SecurityUtils.getSubject().getPrincipal();

 //Step.2 获取导出数据
 List<VxSecretPersonApply> queryList = vxSecretPersonApplyService.list(queryWrapper);
 // 过滤选中数据
 String selections = request.getParameter("selections");
 List<VxSecretPersonApply> vxSecretPersonApplyList = new ArrayList<VxSecretPersonApply>();
 if(oConvertUtils.isEmpty(selections)) {
    vxSecretPersonApplyList = queryList;
 }else {
    List<String> selectionList = Arrays.asList(selections.split(","));
    vxSecretPersonApplyList = queryList.stream().filter(item -> selectionList.contains(item.getId())).collect(Collectors.toList());
 }

 List<VxSecretPersonApplyPage> pageList = new ArrayList<VxSecretPersonApplyPage>();
 TemplateExportParams params = getTemplateParams(property,"涉密岗位登记表","docx");  //获取模板
 Map<String, Object> map = new HashMap<String, Object>();

 SecretPersonExportXls secretPersonExportXls = new SecretPersonExportXls();
 //构造数据
 map = secretPersonExportXls.getSecretRank(vxSecretPersonApplyList, map, sysBaseAPI,secretJobsService);

 // Step.4 easyPoi 导出Excel
 ModelAndView mv = new ModelAndView(new EasypoiTemplateWordView());
 mv.addObject(TemplateWordConstants.FILE_NAME, "涉密岗位登记表");
 mv.addObject(TemplateWordConstants.URL,params.getTemplateUrl());
 mv.addObject(TemplateWordConstants.MAP_DATA,map);
 return mv;

 // 4.导出
 }
```

#### 遇到的问题

java.lang.NoSuchMethodException: org.openxmlformats.schemas.wordprocessingml.x2006.main.impl.CTPictureBaseImpl错误

使用poi对office文档进行操作的时候,出现以下异常

```javascript
java.lang.NoSuchMethodException: org.openxmlformats.schemas.wordprocessingml.x2006.main.impl.CTPictureBaseImpl.<init>(org.apache.xmlbeans.SchemaType, boolean)
	at java.lang.Class.getConstructor0(Unknown Source)
	at java.lang.Class.getDeclaredConstructor(Unknown Source)
	at org.apache.xmlbeans.impl.schema.SchemaTypeImpl.getJavaImplConstructor2(SchemaTypeImpl.java:1817)
	at org.apache.xmlbeans.impl.schema.SchemaTypeImpl.createUnattachedSubclass(SchemaTypeImpl.java:1961)
	at org.apache.xmlbeans.impl.schema.SchemaTypeImpl.createUnattachedNode(SchemaTypeImpl.java:1950)
	at org.apache.xmlbeans.impl.schema.SchemaTypeImpl.createElementType(SchemaTypeImpl.java:1051)
	at org.apache.xmlbeans.impl.values.XmlObjectBase.create_element_user(XmlObjectBase.java:938)
	at org.apache.xmlbeans.impl.store.Xobj.getUser(Xobj.java:1675)
	at org.apache.xmlbeans.impl.store.Cur.getUser(Cur.java:2659)
	at org.apache.xmlbeans.impl.store.Cur.getObject(Cur.java:2652)
	at org.apache.xmlbeans.impl.store.Cursor._getObject(Cursor.java:995)
	at org.apache.xmlbeans.impl.store.Cursor.getObject(Cursor.java:2904)
	at org.apache.poi.xwpf.usermodel.XWPFDocument.onDocumentRead(XWPFDocument.java:162)
	at org.apache.poi.POIXMLDocument.load(POIXMLDocument.java:169)
	at org.apache.poi.xwpf.usermodel.XWPFDocument.<init>(XWPFDocument.java:119)
	at search.utils.POI.readWord(POI.java:169)
	at search.service.ThreadManage.executive(ThreadManage.java:115)
	at search.service.ThreadManage.run(ThreadManage.java:214)
	at java.lang.Thread.run(Unknown Source)
```

实际在项目中已经引入poi-ooxml-schemas-3.17.jar这个包，但是却一直报找不到类的异常。然后查poi官网资料(http://poi.apache.org/faq.html)得知poi提供的那个poi-ooxml-schemas-3.17.jar包是精简版的，为了节省空间，里面放的只有一些常用的模块，所以要引用另外一些功能的话就需要引用完整版的ooxml-schemas.jar包。另外，POI 3.14以上版本对应的完整版的jar包是ooxml-schemas-1.3.jar，这样导入之后果然就好了。 