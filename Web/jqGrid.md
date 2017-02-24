# jqGrid
## 简单介绍
jqGrid 是基于jQuery开发的一个前端动态表格插件，支持主题切换，表格的增删查改，分页等。而且支持目前流行的浏览器，支持主流数据源格式比如：XML，JSON。

## 基本用法
### 创建

需要创建表格主体和表格尾部（用于存放页数和一些排序，增删等按钮）

```Javascript
let jqGridSelector = $('<table id="jqGrid" class="table-fontSize table-font" style="width: 80%;"></table>');
let jqGridPagerSelector = $('<div id="jqGridPager"></div>');
```

### 配置表格

```javascript
jqGridSelector
      .jqGrid({
        datatype: "local",
        height: "80%",
        sortname: '排名',
        sortorder: 'asc',
        colNames: header,
        colModel: colModel,
        rowNum: tableData.data.length,
        rowList: [], // disable page size dropdown
        pgbuttons: false, // disable page control like next, back button
        pgtext: null, // disable pager text like 'Page 0 of 10'
        viewrecords: false, // disable current view record text like 'View 1-10 of 100'
        pager: jqGridPagerSelector,
        altRows: true,
        multiselect: true,
        // multiboxonly: true,
        autowidth: true,
        loadonce: false,
        subGrid: true,
        subGridOptions: {
          plusicon: "glyphicon glyphicon-plus",
          minusicon: "glyphicon glyphicon-minus",
          openicon: "",
          expandOnLoad: false,
          selectOnExpand: false,
          reloadOnExpand: true
        },
        subGridRowExpanded: function (subgrid_id, row_id) {
          console.log('subgrid_id: ', subgrid_id, 'row_id: ', row_id);
          fillChat(jqGridSelector, subgrid_id, row_id);
        },
        subGridRowColapsed: function (subgrid_id, row_id) {
          $('#' + subgrid_id).empty();
        },
        loadError: function (xhr, status, error) {
          console.log(xhr, status, error);
        },
        loadComplete: function (table) {
          console.log('loadComplete: ', table);
        },
        gridComplete: function (a, b) {
          console.log('gridComplete');
          $('#gsh_jqGrid_排名').empty();
          $('#gsh_jqGrid_涨幅').empty();
          $('#gsh_jqGrid_指数').empty();
          $('#gsh_jqGrid_竞品数').empty();
          $('#gsh_jqGrid_中文翻译').empty();
        }
      });

 let localData = [];
      let primaryData = sortTableData(tableData.data);
      for (let i = 0; i < primaryData.length; i++) {
        let dic = primaryData[i];
        let localDic = {
          '关键字': dic.keyword,
          '排名': dic.ranking,
          '涨幅': dic.increase,
          '指数': dic.priority,
          '竞品数': dic.count,
          '中文翻译': dic.cnText,
          'keywordID': dic.keywordID,
          'top': dic.top,
          'topDate': dic.topDate
        };

        localData.push(localDic);
      }
```

 

> * datatype:   配置数据源是本地数组(local)，也可以是JSON，XML，自定义function, JSON和XML都不属于本地数据，所以当设置为JSON或XML是需要提供`url` 选项
> * sortname: 表格默认以哪一列排序
> * colNames：表头各列的名字
> * pager: 设置分页信息显示的html元素
> * altRows: 是否隔行显示，就是一行浅色一行深色
> * multiselect：是否能多行选中
> * multiboxonly：此配置仅当multiselect设置为true时有效果。multiselect为true，点击某行的任何地方都可以选择此行，但是multiboxonly 设置为 true，只有勾选checkbox 才会选中行，点击其他行非checkbox控件将会取消已经选择的行并且选中点击的行
> * subGridOptions: 里面需要配置二级表格打开和关闭的按钮和一些基本选项
> * loadComplete: 数据加载完成时调用
> * gridComplete：表格渲染完成时调用
> * 数据源：数组，每个元素是一个object，object的keys必须与col的名字相同，keys可以比表格的列多，传入一些额外数据有利自定义排序时使用

### 表格列的配置

```javascript
colModel.push({
	name: '关键字',
	index: '关键字',
       	editable: true,
        edittype: 'custom',
        editoptions: {
          custom_element: customAddDialogKeywordElement,
          custom_value: customAddDialogKeywordValue
        },
        sortable: false,
        sorttype: 'string',
        width: 150,
        sopt: ['cn']
      });


function customAddDialogKeywordElement(val, option) {
    let div = $('<div></div>');
    let local = $('<textarea placeholder="本地词" cols="50" id="local_add_jqGrid"></textarea>');
    let en = $('<textarea placeholder="英语词"  cols="50" id="en_add_jqGrid"></textarea>');
    div.append(local);
    div.append('<br>');
    div.append('<br>');
    div.append(en);
    return div;
  }

function customAddDialogKeywordValue(elem) {
    let localTextArea = $(elem).find('#local_add_jqGrid');
    let enTextArea = $(elem).find('#en_add_jqGrid');
    console.log(localTextArea.val(), enTextArea.val());
    return {
      localKeyword: localTextArea.val(),
      enKeyword: enTextArea.val()
    };
  }

let numberSorttype = function (val, rowObj) {
    let grid = $('#jqGrid');
    let sortOrder = grid.jqGrid('getGridParam', 'sortorder');

    if (sortOrder == 'asc') {
      return rowObj.top ? parseInt(val) : parseInt(val) + 9999999;
    }
    return rowObj.top ? parseInt(val) + 9999999 : parseInt(val);
  }
```

> * name：列的唯一索引，必选项
>
> * index：用于排序时的索引，可选
>
> * edittype: 只有editable为true才有效，可以是text, textarea, select, checkbox, password, button, image and file，当点击表格尾部的编辑按钮得到弹窗，这个属性表明弹窗中对应的是什么html元素。使用'custom' 可以自定义元素
>
> * editoptions: 当edittype设置为'custom'后，这里可以配置自定义元素和获取编辑后的数据格式
>
> * sorttype：可以是string, Int等，也可以是定义function，传递当前排序列每行的数据和整行的数据这两个参数，返回一些string或number用于排序
>
> * sopt: 设置搜索该列时默认的搜索规则，`cn` 表示`contains` , 对应关有['eq','ne','lt','le','gt','ge','bw','bn','in','ni','ew','en','cn','nc','nu','nn'] 
>
>    => 
>
>   ['equal','not equal', 'less', 'less or equal','greater','greater or equal', 'begins with','does not begin with','is in','is not in','ends with','does not end with','contains','does not contain','is null','is not null']

### 功能

##### 顶部搜索

```javascript
jqGridSelector.jqGrid('filterToolbar', {
	defaultSearch: 'cn'
});
```

> 只要一设置，每一列只要配置为可搜索的，都会在顶部有个输入框，有时不需要顶部搜索但是需要全局搜索的时候，colModel只能设置为可搜索的，这时就可以在gridComplete里移除掉顶部的搜索框，如上面表格配置的gridComplete

##### 全局搜索增删查改

```javascript
jqGridSelector.jqGrid('navGrid', '#jqGridPager', { //navbar options
        edit: false,
        editicon: 'ace-icon fa fa-pencil blue',
        add: true,
        addicon: 'ace-icon fa fa-plus-circle purple',
        del: true,
        delicon: 'ace-icon fa fa-trash-o red',
        search: true,
        searchicon: 'ace-icon fa fa-search orange',
        refresh: false,
      }, { // edit
        recreateForm: true,
        beforeShowForm: function (e) {
          var form = $(e[0]);
          form.closest('.ui-jqdialog').find('.ui-jqdialog-titlebar').wrapInner('<div class="widget-header" />')
          style_edit_form(form);
        }
      }, {
        //new record form
        //width: 700,
        url: '/keywordMonitor/addKeywordAndFetcher',
        mtype: 'PUT',
        serializeEditData: addNewKeywordCellSerializeData,
        closeAfterAdd: true,
        recreateForm: true,
        viewPagerButtons: false,
        beforeShowForm: function (e) {
          var form = $(e[0]);
          form.closest('.ui-jqdialog').find('.ui-jqdialog-titlebar').wrapInner('<div class="widget-header" />')
          style_edit_form(form);
        },
        afterComplete: afterAddKeywords,
        loadError: function (a, b, c, d) {
          console.log('******', a, b, c, d);
        }
      }, {
        //delete record form
        url: '/keywordMonitor/deleteKeywords',
        mtype: 'PUT',
        serializeDelData: deleteCellSerializeData,
        recreateForm: true,
        beforeShowForm: function (e) {
          var form = $(e[0]);
          if (form.data('styled')) return false;

          form.closest('.ui-jqdialog').find('.ui-jqdialog-titlebar').wrapInner('<div class="widget-header" />')
          style_delete_form(form);

          form.data('styled', true);
        },
        afterComplete: afterDeleteCell
      }, {
        //search form
        recreateForm: true,
        afterShowSearch: function (e) {
          var form = $(e[0]);
          form.closest('.ui-jqdialog').find('.ui-jqdialog-title').wrap('<div class="widget-header" />')
          style_search_form(form);
        },
        afterRedraw: function () {
          style_search_filters($(this));
        },
        multipleSearch: true,
        // sopt: ['cn'],
        closeAfterSearch: true
      });
```

> * 这里的调用 `jqGridSelector.jqGrid('navGrid', '#jqGridPager', options, edittoptions, addoptions, deleteoption, searchOption)` ，表格就会在分页元素上添加上四个按钮，方法中几个按钮的配置不能颠倒，缺少或增多，不需要该功能时也要用`{}` 占位
> * 每个需要操作的options都需要配置配置网络请求参数，如url，mtype，“data”等，也有对应的事件，比如操作完：afterCompleter, data可以是自定义的function， 可以方便的返回自定义的数据格式提交到后台

##### 自定义按钮

```javascript
 jqGridSelector.navButtonAdd('#jqGridPager', {
        caption: "",
        title: "切换关键词类型",
        buttonicon: "fa fa-exchange",
        onClickButton: function () {
          let info = selectdRowInfo(jqGridSelector);

          if (info.keywordIDs.length <= 0) {
            showNotify('请先选择关键字', 'warning');
          } else {
            selectedData = info.rowDatas;
            let keywordStr = info.rowDatas.map(x => x['keyword']).join();
            console.log(info.rowDatas, keywordStr);
            $('#switchKeywordTypeShowingKeywordsDiv').empty();
            $('#switchKeywordTypeShowingKeywordsDiv').html(keywordStr);
            $('#switchKeywordTypeModal').modal('show');
          }
        },
        position: "last"
      });
```

> 自定义按钮需要提供按钮图标， 点击事件和位置（firts or last）



### 文档

[jqGrid](http://www.trirand.com/jqgridwiki/doku.php?id=start)



