<style lang="less">
#app {
  padding: 20px;
  #rMenu {
    position: absolute;
    .rMenuCard {
      width: 180px;
      border: 1px solid #e8eaec;
    }
    .ivu-card-head p,
    .ivu-card-head-inner {
      line-height: 18px;
      height: 18px;
      font-size: 12px;
    }
    .ivu-card-head {
      padding: 0px 10px;
    }
    .ivu-cell {
      padding: 0px 10px;
      .ivu-cell-title {
        font-size: 10px;
        line-height: 20px;
      }
    }
    .ivu-divider-horizontal {
      margin: 1px 0;
    }
  }
}
</style>

<template>
  <div id="app">
    <Card title="zTree-demo">
      <div style="position:relative">
        <div id="areaTree">
          <div class="tree-box">
            <div class="zTreeDemoBackground left">
              <ul id="treeDemo" class="ztree"></ul>
            </div>
          </div>
        </div>
        <!-- 新增节点模态框 -->
        <Modal title="新增节点名称" v-model="bModalNode" :closable="false" @on-ok="handleSave">
          <p style="textAlign:center;">
            <Label>节点名称：</Label>
            <Input v-model="new_node_name" placeholder="请输入节点名称" style="width:300px;"/>
          </p>
        </Modal>
        <!-- 右键菜单 -->
        <div id="rMenu" ref="rMenu" v-show="bShowRightMenu">
          <Card class="rMenuCard" title="菜单" icon="md-pricetags" :padding="0" shadow>
            <CellGroup @on-click="handleRightMenu">
              <Cell name="add" title="新建节点" icon="ios-options">
                <Icon type="ios-add-circle-outline" slot="icon"/>
              </Cell>
              <Cell name="edit" title="重命名节点" icon="ios-options">
                <Icon type="ios-create-outline" slot="icon"/>
              </Cell>
              <Cell name="del" title="删除节点" icon="ios-options">
                <Icon type="ios-remove-circle-outline" slot="icon"/>
              </Cell>
              <Divider/>
              <Cell :name="1" title="添加资产到节点" icon="ios-options">
                <Icon type="md-add-circle" slot="icon"/>
              </Cell>
              <Cell :name="2" title="移动资产到节点" icon="ios-options">
                <Icon type="md-git-compare" slot="icon"/>
              </Cell>
              <Cell :name="3" title="更新节点资产硬件信息" icon="ios-options">
                <Icon type="md-refresh-circle" slot="icon"/>
              </Cell>
              <Cell :name="4" title="测试节点资产可连接性" icon="ios-options">
                <Icon type="md-paw" slot="icon"/>
              </Cell>
            </CellGroup>
          </Card>
        </div>
      </div>
    </Card>
  </div>
</template>
<script>
import jQuery from "jquery/dist/jquery.min.js";

export default {
  data() {
    return {
      // 是否展示右键菜单
      bShowRightMenu: false,
      //   新增节点模态框
      bModalNode: false,
      //   新增节点名称
      new_node_name: "",
      //   原始数据
      objData: [
        { id: 1, pId: 0, name: "节点 1", open: true },
        { id: 11, pId: 1, name: "节点 1-1", open: true },
        { id: 12, pId: 1, name: "节点 1-2", open: true },
        { id: 111, pId: 11, name: "节点 1-1-1", open: true },
        { id: 112, pId: 11, name: "节点 1-1-2", open: true },
        { id: 122, pId: 12, name: "节点 1-2-1", open: true },
        { id: 121, pId: 12, name: "节点 1-2-2", open: true }
      ],
      //   当前节点
      currentNode: {},
      //   ztree对象
      zTree: null,
      //   zTree属性配置
      setting: {
        data: {
          simpleData: {
            enable: true,
            idKey: "id",
            pIdKey: "pId",
            rootPId: 0
          }
        },
        view: {
          // 是否可多选
          selectedMulti: false,
          // 是否显示线
          showLine: true,
          //   是否显示title
          showTitle: true
        },
        edit: {
          // 是否处于编辑状态
          enable: true,
          //   设置 txt 内容是否为全选状态
          editNameSelectAll: true,
          // 设置是否显示删除按钮
          showRemoveBtn: false, //this.showRemoveBtn,
          // 设置是否显示编辑名称按钮
          showRenameBtn: false, //this.showRenameBtn,
          drag: {
            //   拖拽时父节点自动展开是否触发 onExpand 事件回调函数
            autoExpandTrigger: true,
            //   拖拽时, 设置是否允许复制节点
            isCopy: true,
            // 拖拽时, 设置是否允许移动节点
            isMove: true,
            // 拖拽到目标节点时，设置是否允许移动到目标节点前面的操作
            prev: false,
            // 拖拽到目标节点时，设置是否允许移动到目标节点后面的操作
            next: false,
            // 拖拽到目标节点时，设置是否允许成为目标节点的子节点
            inner: true
          }
        },
        callback: {
          // 点击节点
          onClick: this.onClick,
          //   拖拽前
          beforeDrag: this.beforeDrag,
          //   拖拽结束前
          beforeDrop: this.beforeDrop,
          //   重命名前
          beforeRename: this.beforeRename,
          //   重命名
          onRename: this.onRename,
          //   删除前
          beforeRemove: this.beforeRemove,
          //   右击事件
          onRightClick: this.onRightClick
        }
      }
    };
  },
  methods: {
    // 点击节点
    onClick(e, id, node) {
      console.log("onClick", node);
    },
    // 右击菜单选择
    handleRightMenu(name) {
      switch (name) {
        case "add":
          this.bModalNode = true;
          this.new_node_name = "";
          break;
        case "edit":
          this.zTree.editName(this.currentNode);
          break;
        case "del":
          this.zTree.selectNode(this.currentNode);
          this.zTree.removeNode(this.currentNode, true);
          break;

        default:
          break;
      }
      this.bShowRightMenu = false;
    },
    // 右击
    onRightClick(event, treeId, treeNode) {
      this.currentNode = treeNode;
      let x = event.clientX + document.body.scrollLeft - 20,
        y = event.clientY + document.body.scrollTop - 78;

      if (treeNode && this.$refs.rMenu) {
        this.bShowRightMenu = true;
        this.$refs.rMenu.style.top = y + "px";
        this.$refs.rMenu.style.left = x + "px";
      }
    },
    // 保存新增节点
    handleSave() {
      let len = this.currentNode.children
        ? this.currentNode.children.length
        : 0;
      let str = Number(len + 1) + "";
      let res = Number(this.currentNode.id + str);
      this.zTree.addNodes(this.currentNode, {
        id: res,
        pId: this.currentNode.id,
        name: this.new_node_name
      });
      this.success();
    },
    // 修改前
    beforeRename(treeId, treeNode, newName) {
      if (treeNode.name !== newName) {
        this.success();
        return true;
      } else {
        this.error();
        return false;
      }
    },
    //  删除前
    beforeRemove(treeId, treeNode) {
      var mutual = confirm("确认删除 节点 -- " + treeNode.name + " 吗？");
      if (mutual) {
        this.success();
        return true;
      } else {
        this.error();
        return false;
      }
      return true;
    },
    // 操作成功通知栏
    success() {
      this.$Notice.success({
        title: "操作成功！"
      });
    },
    // 操作失败通知栏
    error() {
      this.$Notice.error({
        title: "操作失败！"
      });
    },
    // 拖拽前
    beforeDrag(treeId, treeNodes) {
      // 根节点不允许拖拽
      return !(treeNodes[0].id == 1);
    },
    // 拖拽结束前
    beforeDrop(treeId, treeNodes, targetNode, moveType) {
      if (targetNode) {
        let treeNode = treeNodes[0];
        //   在调用 confirm() 时，将暂停对 JavaScript 代码的执行，在用户作出响应之前，不会执行下一条语句。
        var aa = confirm(
          "您想移动节点：【" +
            treeNode.name +
            "】至【" +
            targetNode.name +
            "】下吗？"
        );
        if (aa) {
          this.zTree.moveNode(targetNode, treeNode, "inner");
          this.success();
          return true;
        } else {
          this.error();
          return false;
        }
      } else {
        this.error();
        return false;
      }
    }
  },
  mounted() {
    $.fn.zTree.init($("#treeDemo"), this.setting, this.objData);
    this.zTree = $.fn.zTree.getZTreeObj("treeDemo");
    let _this = this;
    document.onclick = function(event) {
      if (
        !(
          event.target.id == "rMenu" ||
          $(event.target).parents("#rMenu").length > 0
        )
      ) {
        _this.bShowRightMenu = false;
      }
    };
  }
};
</script>
