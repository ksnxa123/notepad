<template>
  <div id="lesson">
    <el-container>
      <el-header style="font-size:35px;font-weight:Bolder">在线笔记本</el-header>
      <el-header style="height:45px">
        <el-button
          :title="noteNumber"
          type="text"
          icon="el-icon-document-add"
          style="margin-right:15px"
          v-on:click="centerDialogVisible=true"
        >添加笔记</el-button>

        <el-dialog title="添加笔记" :visible.sync="centerDialogVisible" width="50%" center>
          <el-form ref="form" :model="form" label-width="80px">
            <el-form-item label="标题">
              <el-input v-model="form.title"></el-input>
            </el-form-item>

            <el-form-item label="内容">
              <el-input type="textarea" v-model="form.content"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="addNote">添加</el-button>
              <el-button>取消</el-button>
            </el-form-item>
          </el-form>
        </el-dialog>

        <el-button
          type="text"
          icon="el-icon-document"
          style="margin-right:15px"
          v-on:click="centerDialogVisible1=true"
        >修改笔记</el-button>

        <el-dialog title="修改笔记" :visible.sync="centerDialogVisible1" width="50%" center>
          <template v-if="selectedNote">
            <label>
              标题
              <el-input v-model="selectedNote.title"></el-input>
            </label>

            <label>
              内容
              <el-input type="textarea" v-model="selectedNote.content"></el-input>
            </label>
          </template>
        </el-dialog>

        <el-button
          type="text"
          icon="el-icon-document-delete"
          style="margin-right:15px"
          @click="removeNote"
        >删除笔记</el-button>
        <el-button type="text" @click="preferenceNote" v-if="selectedNote">
          <div v-if="selectedNote.preference">
            <i class="el-icon-star-on"></i>推荐
          </div>
          <div v-else>
            <i class="el-icon-star-off"></i>取消推荐
          </div>
        </el-button>
      </el-header>
      <el-container>
        <el-aside width="300px">
          <el-menu default-active="2" class="el-menu-vertical-demo">
            <el-menu-item
              :index="index+''"
              v-for="(item,index) in this.sortedNotes"
              :key="index"
              @click="selectNote(item)"
            >
              <span slot="title">
                <i class="el-icon-star-on" v-if="item.preference" />
                <i class="el-icon-star-off" v-else />
                {{item.title}}
              </span>
            </el-menu-item>
            <!-- <el-menu-item index="3">
              <span slot="title">笔记2...</span>
            </el-menu-item>
            <el-menu-item index="4">
              <span slot="title">笔记3...</span>
            </el-menu-item>
            <el-menu-item index="5">
              <span slot="title">笔记4...</span>
            </el-menu-item>-->
          </el-menu>
        </el-aside>
        <el-main>
          <el-container>
            <template v-if="selectedNote">
              <!-- <el-aside width="200px" style="background:lightgray">
                <textarea v-model="selectedNote.content"></textarea>
              </el-aside> -->
              <el-main v-html="notePreview"></el-main>
            </template>
          </el-container>
        </el-main>
      </el-container>
      <el-footer>
        <template v-if="selectedNote">
        <label>创建时间：{{selectedNote.createTime|dateFormat}}</label>
        <label>字数：{{words}}</label>
        </template>
      </el-footer>
    </el-container>
  </div>
</template>

<script>
//npm run build 生成项目包
//创建vue.config.js 配置路径，如果上传网站根目录则不需要此步骤
//上传服务器或虚拟空间在互联网上浏览自己创建的项目

import marked from "marked";
import moment from "moment";

export default {
  data() {
    return {
      notes: JSON.parse(localStorage.getItem("notes")) || [],
      centerDialogVisible: false,
      centerDialogVisible1: false,
      //content: this.content=localStorage.getItem('content')||'你可以在文本输入框当中输入内容！',
      preference: false,
      selectId: localStorage.getItem("selected-id"),
      form: {
        title: "",
        content: ""
      }
    };
  },
  methods: {
    test() {
      alert("ok");
    },
    addNote() {
      const time = Date.now();
      const note = {
        id: String(time),
        title: this.form.title,
        content: this.form.content,
        createTime: time,
        preference: false
      };
      //添加到笔记列表当中
      this.notes.push(note);
      this.form.title = "";
      this.form.content = "";
      alert(JSON.stringify(this.notes));
    },
    selectNote(note) {
      this.selectId = note.id;
      console.log(this.selectId);
    },
    // saveNote() {
    //   console.log("笔记已保存：" + this.content);
    //   localStorage.setItem("content", this.content);
    // },
    saveNotes() {
      console.log("笔记已保存：" + new Date());
      localStorage.setItem("notes", JSON.stringify(this.notes));
    },
    removeNote() {
      if (this.selectedNote && confirm("您确认删除该笔记吗?")) {
        //将选中的笔记从列表中移除
        const index = this.notes.indexOf(this.selectedNote);
        if (index != -1) {
          this.notes.splice(index, 1);
        }
      }
    },
    preferenceNote() {
      this.selectedNote.preference = !this.selectedNote.preference;
      //this.selectedNote.preference=this.selectedNote.preference^true;
      //this.selectedNote.preference^=true;
    },
    wordsCount(str) {
      //用js计算length时不管是中文还是英文,都记为1,但本代码中文计算为2
      var str_len = str.length;
      var str_length = 0;
      var a;
      for (var i = 0; i < str_len; i++) {
        a = str.charAt(i);
        str_length++;
        // if (escape(a).length > 4) {
        //   //中文字符的长度经编码之后大于4
        //   str_length++;
        // }
      }
      return str_length;
    }
  },
  watch: {
    //   content:{
    //       handler(val,oldval){
    //       console.log("新值："+val+"+老的值："+oldval)
    //        localStorage.setItem("content", val);
    //       },
    //       //immediate:true
    //   }
    // content(val, oldval) {
    //   console.log("新值：" + val + "+老的值：" + oldval);
    //   localStorage.setItem("content", val);
    // }
    //immediate:true
    //content:'saveNote'

    notes: {
      handler: "saveNotes",
      deep: true
    },
    selectId(val) {
      localStorage.setItem("selected-id", val);
    }
  },
  computed: {
    notePreview() {
      return this.selectedNote ? marked(this.selectedNote.content) : "";
    },
    noteNumber() {
      return "目前有" + this.notes.length + "条笔记";
    },
    selectedNote() {
      //获取与selectedId匹配的笔记
      return this.notes.find(note => note.id === this.selectId);
    },
    //2个步骤对列表进行排序
    //1 根据创建时间对所有笔记排序，
    //2 然后把收藏的笔记排列在前面
    //使用标准数组方法sort，非常方便该方法接收1个参数（接收两个参数的方法），对这两个参数做比较 比较结果是一个数，
    //--------------
    //0 两个参数相等，
    //-1 第一个参数在第二个参数前面
    //1 第一个参数在第二个参数后面
    //--------------------------------
    //创建计算属性，可通过vue机制对排序自动更新和保存
    sortedNotes() {
      return (
        this.notes
          .slice() //从已有的数组中返回选定的元素
          //1 通过减法来得到，对两条笔记的创建时间来比较，如a在b之前，为正，否则为负值
          .sort((a, b) => a.createTime - b.createTime)
          .sort((a, b) =>
            //2 使用两个三元操作(什么叫三元操作？)
            //如果a,b两个笔记都已收藏，则不改变位置
            //如仅收藏a 返回负值，将a排至b前面
            //如仅收藏b 返回正值，将b排至a前面
            a.preference === b.preference ? 0 : a.preference ? -1 : 1
          )
      );
    },
    words() {
      return this.wordsCount(this.selectedNote.content);
    }
  },
  filters: {
    dateFormat(date) {
      return moment(date).format("YYYY年MM月DD HH:mm:ss");
    }
  }
  //   created(){
  //      this.content=localStorage.getItem('content')||'你可以在文本输入框当中输入内容！'
  //   }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus">
.el-header {
  background-color: #B3C0D1;
  color: #333;
  text-align: left;
  line-height: 60px;
}

.el-footer {
  background-color: #B3C0D1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background-color: #999;
  color: #333;
  text-align: left;
  line-height: 200px;
}

.el-main {
  background-color: #9f9f9f;
  color: #333;
  text-align: left;
  line-height: 30px;
}

body > .el-container {
  margin-bottom: 40px;
}

.el-container:nth-child(5) .el-aside, .el-container:nth-child(6) .el-aside {
  line-height: 260px;
}

.el-container:nth-child(7) .el-aside {
  line-height: 320px;
}
</style>