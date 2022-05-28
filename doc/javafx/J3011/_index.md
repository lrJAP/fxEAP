---
title: "TableView"
weight: 3011
---



Form、Table组件是lrEAP中UI模式化开发重要的基础组件。所以我们在开发过程中投入了相当的精力及关注，力求好用、美观、完善。但JavaFX经过这么多年的发展，虽然看上去基础组件比较齐全了，但是在细节方面，还是存在明显的不足，特别是在企业级应用对基础组件的要求方面。例如，Table组件开发过程中使用TableView所遇到的问题：

1.	在双击单元格进行编辑时，JavaFX采用在当前单元格中建立TextField替换Label的方式实现单元格的编辑功能，默认情况下会导致当前单元格的行高产生变化。这种非常不友好的行为表现在如此基础并且重要的组件中，让我们感觉无法理解并且难以容忍。
2.	TableCell对扩展类型的支持不到位。默认情况下只支持一些简单类型(如字符、数字等)的直接录入，没有提供对字体、颜色、参照等弹出单元格较好的处理方式。
3.	TableCell的Commit机制比较古怪。对于一般的TableCell来说，必须在当前单元格中进行“回车”之后，对当前单元的修改才会被真正提交。但是我们操作人员一般的习惯，是点击单元格，输入或选择内容，然后点击（或是按Tab键）另一个需要编辑单元格——特别是处理参照类型的单元格时。但是非常不幸，TableView默认情况下进行上述操作时，会回滚对当前单元格所做的变更。

基于JavaFX TableView扩展生成新的组件，主要演示解决以下问题：

1. 无数据时，不绘制表格线。通过CSS实现。

2. 增加“行号”列。
3. 通过自定义的TextFieldTableCell，解决TableCell失去焦点时自动CancelEdit的问题。
4. 实现自适应列宽

## HelloLRTableView

```java
package com.lirong.eap.test.site.j1004;

import com.lirong.eap.platform.base.client.utils.MessageDialogUtil;
import com.lirong.eap.platform.base.client.utils.Metadata2TableColumnUtil;
import com.lirong.eap.platform.client.javafx.control.pane.MasterDetailPane;
import com.lirong.eap.platform.client.javafx.control.view.LRTableView;
import com.lirong.eap.test.site.j1004.utils.DataUtils;
import com.lirong.eap.test.site.j1004.vo.TestFXVO;
import javafx.application.Application;
import javafx.collections.ObservableList;
import javafx.geometry.Insets;
import javafx.geometry.Side;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.TextArea;
import javafx.scene.layout.BorderPane;
import javafx.scene.layout.HBox;
import javafx.stage.Stage;

import static com.lirong.eap.platform.base.pub.utils.ValueObjectUtils.LINE_SEP;

/**
 * <p>Title: LiRong Java Enterprise Application Platform</p>
 * <p>Description: TestVO在LRTableView中显示、编辑。用于演示LRTableView对TestVO中各位数据类型的支持。 </p>
 * Copyright: CorpRights lrJAP.com<br>
 * Company: lrJAP.com<br>
 *
 * @author jianjun.yu
 * @version 3.0.0-SNAPSHOT
 * @date 2022-05-20
 * @since 1.0.0-SNAPSHOT
 */
public class HelloLRTableView extends Application {

    public static void main(String[] args) {

        launch(args);
    }

    @Override
    public void start(Stage primaryStage) throws Exception {

        LRTableView tableView = new LRTableView();
        tableView.setEditable(true);
        tableView.setShowRowNumberColumn(true);
        // create column metadata with PropertyVO
        tableView.getColumns().addAll(Metadata2TableColumnUtil.createTableColumnsByMetadata(DataUtils.createPropertyItem()));
        // must after add columns
        tableView.setShowMultiSelectColumn(true);

        TextArea textConsole = new TextArea();
        textConsole.setPrefHeight(360);

        Button buttonLoadData = new Button("LoadData");
        buttonLoadData.setPrefSize(75, 30);
        // load data
        buttonLoadData.setOnAction(action -> tableView.getItems().setAll(DataUtils.createDataList(100)));

        Button buttonPrintSelected = new Button("PrintSelected");
        buttonPrintSelected.setPrefSize(120, 30);
        // print data
        buttonPrintSelected.setOnAction(action -> {
            textConsole.clear();
            ObservableList<TestFXVO> listSelected = tableView.getSelectionModel().getSelectedItems();
            if (listSelected.isEmpty()) {
                MessageDialogUtil.showMessage("You selected nothing.");
            } else {
                for (TestFXVO testFXVO : listSelected) {
                    textConsole.appendText(testFXVO.toString());
                    textConsole.appendText(LINE_SEP);
                }
            }
        });

        HBox toolbar = new HBox();
        toolbar.setPadding(new Insets(10));
        toolbar.setSpacing(10);
        toolbar.getChildren().addAll(buttonLoadData, buttonPrintSelected);

        MasterDetailPane masterDetailPane = new MasterDetailPane(Side.BOTTOM);
        masterDetailPane.setDividerPosition(0.7);
        masterDetailPane.setMasterNode(tableView);
        masterDetailPane.setDetailNode(textConsole);

        BorderPane container = new BorderPane();
        container.setTop(toolbar);
        container.setCenter(masterDetailPane);

        Scene scene = new Scene(container, 1024, 768);
        primaryStage.setTitle("LRTableView Test");
        primaryStage.setScene(scene);
        primaryStage.show();
    }
}
```

## DataUtils

```java
package com.lirong.eap.test.site.j1004.utils;

import com.lirong.eap.platform.base.pub.vo.PDMDomainConstant;
import com.lirong.eap.platform.metadata.pub.enumtype.CodeNameFieldFlagEnum;
import com.lirong.eap.platform.metadata.pub.vo.PropertyVO;
import com.lirong.eap.platform.metadata.pub.vo.reference.DisplayCategoryReferenceVO;
import com.lirong.eap.test.site.j1004.vo.TestFXVO;
import com.lirong.eap.test.site.j1004.vo.TestVO;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;

import java.util.ArrayList;
import java.util.List;

/**
 * <p>Title: LiRong Java Enterprise Application Platform</p>
 * <p>Description:  </p>
 * Copyright: CorpRights lrJAP.com<br>
 * Company: lrJAP.com<br>
 *
 * @author jianjun.yu
 * @version 3.0.0-SNAPSHOT
 * @date 2022-05-20
 * @since 1.0.0-SNAPSHOT
 */
public final class DataUtils {

    private DataUtils() {

    }

    public static ObservableList<TestFXVO> createDataList(int count) {

        ObservableList<TestFXVO> list = FXCollections.observableArrayList();

        for (int i = 0; i < count; i++) {
            TestFXVO fxVO = new TestFXVO(new TestVO(i + 1));
            list.add(fxVO);
        }

        return list;
    }

    /**
     * 手工创建属性元数据信息
     */
    public static List<PropertyVO> createPropertyItem() {

        // FormView Show Group: Normal
        DisplayCategoryReferenceVO normal = new DisplayCategoryReferenceVO("1", "Normal", "常规");
        // FormView Show Group: Audit
        DisplayCategoryReferenceVO audit = new DisplayCategoryReferenceVO("2", "Audit", "审计");

        List<PropertyVO> list = new ArrayList<>();

        PropertyVO testCodeProperty = new PropertyVO();
        testCodeProperty.setPropertyCode("testCode");
        testCodeProperty.setPropertyName("Code");
        testCodeProperty.setPrimaryKeyFlag(true);
        testCodeProperty.setPdmDomain(PDMDomainConstant.CODE32_TYPE_DOMAIN);
        testCodeProperty.setCodeNameFieldFlag(CodeNameFieldFlagEnum.CODE_FIELD);
        testCodeProperty.setOrderNumber(1);
        testCodeProperty.setJavaDataType("java.lang.String");
        testCodeProperty.setJavaShortDataType("String");
        testCodeProperty.setDbDataType("VARCHAR");
        testCodeProperty.setMaxLength(32);
        testCodeProperty.setNullAllowFlag(false);
        testCodeProperty.setEditableFlag(true);
        testCodeProperty.setSortAble(true);
        testCodeProperty.setAutoresizeColumn(true);
        testCodeProperty.setFormShowFlag(true);
        testCodeProperty.setPkDisplayCategory(normal);
        list.add(testCodeProperty);

        PropertyVO testNameProperty = new PropertyVO();
        testNameProperty.setPropertyCode("testName");
        testNameProperty.setPropertyName("Name");
        testNameProperty.setPdmDomain(PDMDomainConstant.NAME32_TYPE_DOMAIN);
        testNameProperty.setCodeNameFieldFlag(CodeNameFieldFlagEnum.NAME_FIELD);
        testNameProperty.setOrderNumber(2);
        testNameProperty.setJavaDataType("java.lang.String");
        testNameProperty.setJavaShortDataType("String");
        testNameProperty.setDbDataType("VARCHAR");
        testNameProperty.setMaxLength(100);
        testNameProperty.setNullAllowFlag(false);
        testNameProperty.setEditableFlag(true);
        testNameProperty.setSortAble(true);
        testNameProperty.setAutoresizeColumn(true);
        testNameProperty.setFormShowFlag(true);
        testNameProperty.setPkDisplayCategory(normal);
        list.add(testNameProperty);

        PropertyVO testByteProperty = new PropertyVO();
        testByteProperty.setPropertyCode("testByte");
        testByteProperty.setPropertyName("Byte");
        testByteProperty.setOrderNumber(3);
        testByteProperty.setJavaDataType("java.lang.Byte");
        testByteProperty.setJavaShortDataType("Byte");
        testByteProperty.setDbDataType("DECIMAL");
        testByteProperty.setMaxLength(160);
        testByteProperty.setNullAllowFlag(false);
        testByteProperty.setEditableFlag(true);
        testByteProperty.setSortAble(true);
        testByteProperty.setAutoresizeColumn(true);
        testByteProperty.setFormShowFlag(true);
        testByteProperty.setPkDisplayCategory(normal);
        list.add(testByteProperty);

        PropertyVO testShortProperty = new PropertyVO();
        testShortProperty.setPropertyCode("testShort");
        testShortProperty.setPropertyName("Short");
        testShortProperty.setOrderNumber(4);
        testShortProperty.setJavaDataType("java.lang.Short");
        testShortProperty.setJavaShortDataType("Short");
        testShortProperty.setDbDataType("DECIMAL");
        testShortProperty.setMaxLength(160);
        testShortProperty.setNullAllowFlag(false);
        testShortProperty.setEditableFlag(true);
        testShortProperty.setSortAble(true);
        testShortProperty.setAutoresizeColumn(true);
        testShortProperty.setFormShowFlag(true);
        testShortProperty.setPkDisplayCategory(normal);
        list.add(testShortProperty);

        PropertyVO testLongProperty = new PropertyVO();
        testLongProperty.setPropertyCode("testLong");
        testLongProperty.setPropertyName("Long");
        testLongProperty.setOrderNumber(5);
        testLongProperty.setJavaDataType("java.lang.Long");
        testLongProperty.setJavaShortDataType("Long");
        testLongProperty.setDbDataType("DECIMAL");
        testLongProperty.setMaxLength(160);
        testLongProperty.setNullAllowFlag(false);
        testLongProperty.setEditableFlag(true);
        testLongProperty.setSortAble(true);
        testLongProperty.setAutoresizeColumn(true);
        testLongProperty.setFormShowFlag(true);
        testLongProperty.setPkDisplayCategory(normal);
        list.add(testLongProperty);

        PropertyVO testAgeProperty = new PropertyVO();
        testAgeProperty.setPropertyCode("testInteger");
        testAgeProperty.setPropertyName("Integer");
        testAgeProperty.setOrderNumber(6);
        testAgeProperty.setJavaDataType("java.lang.Integer");
        testAgeProperty.setJavaShortDataType("Integer");
        testAgeProperty.setDbDataType("INT");
        testAgeProperty.setMaxLength(100);
        testAgeProperty.setNullAllowFlag(false);
        testAgeProperty.setEditableFlag(true);
        testAgeProperty.setSortAble(true);
        testAgeProperty.setAutoresizeColumn(true);
        testAgeProperty.setFormShowFlag(true);
        testAgeProperty.setPkDisplayCategory(normal);
        list.add(testAgeProperty);

        PropertyVO testSalaryProperty = new PropertyVO();
        testSalaryProperty.setPropertyCode("testDecimal");
        testSalaryProperty.setPropertyName("Decimal");
        testSalaryProperty.setOrderNumber(7);
        testSalaryProperty.setJavaDataType("java.math.BigDecimal");
        testSalaryProperty.setJavaShortDataType("BigDecimal");
        testSalaryProperty.setPrecise(2);
        testSalaryProperty.setDbDataType("DECIMAL");
        testSalaryProperty.setMaxLength(100);
        testSalaryProperty.setNullAllowFlag(false);
        testSalaryProperty.setEditableFlag(true);
        testSalaryProperty.setSortAble(true);
        testSalaryProperty.setAutoresizeColumn(true);
        testSalaryProperty.setFormShowFlag(true);
        testSalaryProperty.setPkDisplayCategory(normal);
        list.add(testSalaryProperty);

        PropertyVO testBooleanProperty = new PropertyVO();
        testBooleanProperty.setPropertyCode("testBoolean");
        testBooleanProperty.setPropertyName("Boolean");
        testBooleanProperty.setOrderNumber(8);
        testBooleanProperty.setJavaDataType("java.lang.Boolean");
        testBooleanProperty.setJavaShortDataType("Boolean");
        testBooleanProperty.setDbDataType("VARCHAR");
        testBooleanProperty.setMaxLength(100);
        testBooleanProperty.setNullAllowFlag(false);
        testBooleanProperty.setEditableFlag(true);
        testBooleanProperty.setSortAble(true);
        testBooleanProperty.setAutoresizeColumn(true);
        testBooleanProperty.setFormShowFlag(true);
        testBooleanProperty.setPkDisplayCategory(normal);
        list.add(testBooleanProperty);

        PropertyVO testBirthdayProperty = new PropertyVO();
        testBirthdayProperty.setPropertyCode("testDate");
        testBirthdayProperty.setPropertyName("Date");
        testBirthdayProperty.setOrderNumber(9);
        testBirthdayProperty.setJavaDataType("java.time.LocalDate");
        testBirthdayProperty.setJavaShortDataType("LocalDate");
        testBirthdayProperty.setDbDataType("VARCHAR");
        testBirthdayProperty.setMaxLength(160);
        testBirthdayProperty.setNullAllowFlag(false);
        testBirthdayProperty.setEditableFlag(true);
        testBirthdayProperty.setSortAble(true);
        testBirthdayProperty.setAutoresizeColumn(true);
        testBirthdayProperty.setFormShowFlag(true);
        testBirthdayProperty.setPkDisplayCategory(audit);
        list.add(testBirthdayProperty);

        PropertyVO testTimeProperty = new PropertyVO();
        testTimeProperty.setPropertyCode("testTime");
        testTimeProperty.setPropertyName("Time");
        testTimeProperty.setOrderNumber(10);
        testTimeProperty.setJavaDataType("java.time.LocalTime");
        testTimeProperty.setJavaShortDataType("LocalTime");
        testTimeProperty.setDbDataType("VARCHAR");
        testTimeProperty.setMaxLength(160);
        testTimeProperty.setNullAllowFlag(false);
        testTimeProperty.setEditableFlag(true);
        testTimeProperty.setSortAble(true);
        testTimeProperty.setAutoresizeColumn(true);
        testTimeProperty.setFormShowFlag(true);
        testTimeProperty.setPkDisplayCategory(audit);
        list.add(testTimeProperty);

        PropertyVO testDataTimeProperty = new PropertyVO();
        testDataTimeProperty.setPropertyCode("testDateTime");
        testDataTimeProperty.setPropertyName("DateTime");
        testDataTimeProperty.setOrderNumber(11);
        testDataTimeProperty.setJavaDataType("java.time.LocalDateTime");
        testDataTimeProperty.setJavaShortDataType("LocalDateTime");
        testDataTimeProperty.setDbDataType("VARCHAR");
        testDataTimeProperty.setMaxLength(160);
        testDataTimeProperty.setNullAllowFlag(false);
        testDataTimeProperty.setEditableFlag(true);
        testDataTimeProperty.setSortAble(true);
        testDataTimeProperty.setAutoresizeColumn(true);
        testDataTimeProperty.setFormShowFlag(true);
        testDataTimeProperty.setPkDisplayCategory(audit);
        list.add(testDataTimeProperty);

        PropertyVO testDataStatusProperty = new PropertyVO();
        testDataStatusProperty.setPropertyCode("valueObjectStatus");
        testDataStatusProperty.setPropertyName("Enum");
        testDataStatusProperty.setOrderNumber(12);
        testDataStatusProperty.setPdmDomain(PDMDomainConstant.ENUM_TYPE_DOMAIN);
        testDataStatusProperty.setJavaDataType("com.lirong.eap.platform.base.pub.enumtype.DataStatusEnum");
        testDataStatusProperty.setJavaShortDataType("DataStatusEnum");
        testDataStatusProperty.setDbDataType("VARCHAR");
        testDataStatusProperty.setMaxLength(160);
        testDataStatusProperty.setNullAllowFlag(false);
        testDataStatusProperty.setEditableFlag(true);
        testDataStatusProperty.setSortAble(true);
        testDataStatusProperty.setAutoresizeColumn(true);
        testDataStatusProperty.setEnumFlag(true);
        testDataStatusProperty.setFormShowFlag(true);
        testDataStatusProperty.setPkDisplayCategory(audit);
        list.add(testDataStatusProperty);

        PropertyVO testUserReferenceProperty = new PropertyVO();
        testUserReferenceProperty.setPropertyCode("pkUser");
        testUserReferenceProperty.setPropertyName("Reference");
        testUserReferenceProperty.setOrderNumber(13);
        testUserReferenceProperty.setPdmDomain(PDMDomainConstant.ENUM_TYPE_DOMAIN);
        testUserReferenceProperty.setJavaDataType("com.lirong.eap.platform.sm.pub.vo.reference.UserReferenceVO");
        testUserReferenceProperty.setJavaShortDataType("UserReferenceVO");
        testUserReferenceProperty.setDbDataType("VARCHAR");
        testUserReferenceProperty.setMaxLength(160);
        testUserReferenceProperty.setNullAllowFlag(false);
        testUserReferenceProperty.setEditableFlag(true);
        testUserReferenceProperty.setSortAble(true);
        testUserReferenceProperty.setAutoresizeColumn(true);
        testUserReferenceProperty.setReferenceFlag(true);
        testUserReferenceProperty.setFormShowFlag(true);
        testUserReferenceProperty.setPkDisplayCategory(audit);

        list.add(testUserReferenceProperty);

        return list;
    }
}
```

## 运行效果

<video controls src="01.mp4" poster="" autoplay>
</video>
