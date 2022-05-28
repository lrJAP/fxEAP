---
title: "FormView"
weight: 3010
---

## HelloFormView

```java
package com.lirong.eap.test.site.j1004;

import com.lirong.eap.platform.client.javafx.control.pane.MasterDetailPane;
import com.lirong.eap.platform.client.javafx.control.view.FormView;
import com.lirong.eap.test.site.j1004.utils.DataUtils;
import com.lirong.eap.test.site.j1004.vo.TestFXVO;
import com.lirong.eap.test.site.j1004.vo.TestVO;
import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.geometry.Side;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.TextArea;
import javafx.scene.layout.BorderPane;
import javafx.scene.layout.HBox;
import javafx.stage.Stage;

/**
 * <p>Title: LiRong Java Enterprise Application Platform</p>
 * <p>Description: TestVO在FormView中显示、编辑。用于演示FormView对TestVO中各位数据类型的支持。 </p>
 * Copyright: CorpRights lrJAP.com<br>
 * Company: lrJAP.com<br>
 *
 * @author jianjun.yu
 * @version 3.0.0-SNAPSHOT
 * @date 2022-05-20
 * @since 1.0.0-SNAPSHOT
 */
public class HelloFormView extends Application {

    public static void main(String[] args) {

        launch(args);
    }

    @Override
    public void start(Stage primaryStage) throws Exception {

        TestFXVO vo = new TestFXVO(new TestVO(1));

        FormView formView = new FormView(3);
        // disable form edit
        formView.setFormEditEnable(false);
        // create ValueObject metadata for FormView
        formView.setListPropertyVO(DataUtils.createPropertyItem());

        TextArea textConsole = new TextArea();

        Button buttonLoadData = new Button("LoadData");
        buttonLoadData.setPrefSize(75, 25);
        // set data to FormView
        buttonLoadData.setOnAction(action -> {
            formView.setValueObject(vo);
        });

        Button buttonEditData = new Button("EditData");
        buttonEditData.setPrefSize(75, 30);
        // enable edit data
        buttonEditData.setOnAction(action -> {
            if (formView.getValueObject() == null) {
                formView.setValueObject(vo);
            }
            // disable form edit
            formView.setFormEditEnable(true);
        });

        Button buttonPrintData = new Button("PrintData");
        buttonPrintData.setPrefSize(75, 30);
        // print data
        buttonPrintData.setOnAction(action -> {
            if (formView.getValueObject() == null) {
                formView.setValueObject(vo);
            }
            // print data and disable form edit
            textConsole.appendText(vo.toString());
            formView.setFormEditEnable(false);
        });

        HBox box = new HBox();
        box.setSpacing(10);
        box.setPadding(new Insets(10));
        box.getChildren().addAll(buttonLoadData, buttonEditData, buttonPrintData);

        MasterDetailPane centerPane = new MasterDetailPane(Side.BOTTOM);
        centerPane.setDividerPosition(0.7);
        centerPane.setMasterNode(formView);
        centerPane.setDetailNode(textConsole);

        BorderPane container = new BorderPane();
        container.setTop(box);
        container.setCenter(centerPane);

        Scene scene = new Scene(container, 1024, 768);
        primaryStage.setTitle("FormView Test");
        primaryStage.setScene(scene);
        primaryStage.show();
    }
}
```

## 运行效果

<video controls src="01.mp4" poster="" autoplay>
</video>
