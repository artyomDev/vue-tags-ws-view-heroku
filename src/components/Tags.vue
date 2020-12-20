<template>
  <h1>{{ msg }}</h1>
  <br/>
  <br/>
  <div class="container">

    <!-- Add tag start -->
    <div class="row add-tag">
      <div class="col col-lg-10 col-md-10 col-sm-8 col-xs-12">
        <input type="text" class="form-control" placeholder="Add Tag" id="addtagname" />
      </div>
      <div class="col col-lg-2 col-md-2 col-sm-4 col-xs-12">
        <button class="btn btn-primary" v-on:click="addTag">Add</button>
      </div>
    </div>
    <!-- Add tag end -->

    <!-- Tag list start -->
    <div class="row tag-label" v-for="(tag, index) in tags" :key="index" v-on:click="selectTag(index, tag.color)">
      <div class="col col-lg-1 col-md-1 col-sm-2 col-xs-4">
        <div class="dot-img-wrapper"><span class="dot-img" v-bind:style="{ backgroundColor: tag.color }"></span></div>
      </div>
      <div class="col col-lg-7 col-md-7 col-sm-4 col-xs-8 text-left">
        <div class="tag-name-wrapper"><span class="tag-name">{{tag.name}}</span></div>
      </div>
      <div class="col col-lg-2 col-md-2 col-sm-3 col-xs-6">
        <button type="button" class="btn btn-warning" data-toggle="modal" data-target="#editTag" v-on:click="editTag(tag.name, tag.id)">Edit</button>
      </div>
      <div class="col col-lg-2 col-md-2 col-sm-3 col-xs-6">
        <button type="button" class="btn btn-danger" data-toggle="modal" data-target="#deleteTag" v-on:click="deleteTag(tag.id)">Delete</button>
      </div>
    </div>
    <!-- Tag list end -->

    <!-- Edit tag modal start -->
    <div class="modal fade" id="editTag" tabindex="-1" role="dialog" aria-labelledby="editTagLabel" aria-hidden="true">
      <div class="modal-dialog" role="document">
          <div class="modal-content">
          <div class="modal-header">
              <h5 class="modal-title" id="editTagLabel">Edit Tag</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
              </button>
          </div>
          <div class="modal-body text-left">
            <br/>
            <label>Tag name</label>
            <input type="text" class="form-control" placeholder="Add Tag" id="editTagValue" />
            <br/>
          </div>
          <div class="modal-footer">
              <button type="button" class="btn btn-danger btn-modal-close" data-dismiss="modal">Cancel</button>
              <button type="button" class="btn btn-primary" id="saveTag" v-on:click="confirmEdit">Save changes</button>
          </div>
          </div>
      </div>
    </div>
    <!-- Edit tag modal end -->

    <!-- Delete tag modal start -->
    <div class="modal fade" id="deleteTag" tabindex="-1" role="dialog" aria-labelledby="deleteTagLabel" aria-hidden="true">
      <div class="modal-dialog" role="document">
          <div class="modal-content">
          <div class="modal-header">
              <h5 class="modal-title" id="deleteTagLabel">Confirm Delete</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
              </button>
          </div>
          <div class="modal-body">
            <p>Are you sure to delete this tag?</p>
          </div>
          <div class="modal-footer">
              <button type="button" class="btn btn-danger btn-modal-close" data-dismiss="modal">Cancel</button>
              <button type="button" class="btn btn-primary" id="deleteConfirmTag" v-on:click="confirmDelete">Confrim</button>
          </div>
          </div>
      </div>
    </div>
    <!-- Delete tag modal end -->
    
  </div>    
</template>

<script>
  import $ from 'jquery';
  const backendurl = require("../backendconfig.js").url;
  const wsurl = require("../backendconfig.js").wsurl;
  var Socket = new WebSocket(wsurl + "/getall");
  

  export default {
    name: 'Tags',
    props: {
      msg: String
    },
    data() {
      return {
        tags: []
      }
    },
    mounted() {
      var self = this;

      Socket.onopen = function() {
        Socket.send("get all tags");
      };

      setInterval(function(){Socket.send("get all tags")}, 3000);
      
      Socket.onmessage = function (evt) { 
        var received_msg = evt.data;
        // console.log(received_msg);
        var tags = JSON.parse(received_msg);
        self.tags = tags;
      };
    },
    methods: {
      addTag() {
        var randomColor = Math.floor(Math.random()*16777215).toString(16);
        randomColor = "#" + randomColor;
        var tagname = $("#addtagname").val();
        $.post(backendurl + "/add", {color: randomColor, name: tagname}).then(function(){
          $("#addtagname").val("");
          
          Socket.send("get all tags");
            $(".tag-label").css("border-left", "none");
            $(".tag-label").css("background-color", "white");
        });
      },
      selectTag(index, color) {
        $(".tag-label").css("border-left", "none");
        $(".tag-label").css("background-color", "white");
        $(".tag-label").eq(index).css("border-left", "3px solid " + color);
        $(".tag-label").eq(index).css("background-color", "#eee");
      },
      editTag(name, tagid) {
        $("#editTagValue").val(name);
        $("#saveTag").attr("tagid", tagid);
      },
      confirmEdit() {
        var tagid = $("#saveTag").attr("tagid");
        var tagname = $("#editTagValue").val();
        $.post(backendurl + "/update", {id: tagid, name: tagname}).then(function(){
          $(".btn-modal-close").click();
          Socket.send("get all tags");
            $(".tag-label").css("border-left", "none");
            $(".tag-label").css("background-color", "white");
        });
      },
      deleteTag(tagid) {
        $("#deleteConfirmTag").attr("tagid", tagid);
      },
      confirmDelete() {
        var tagid = $("#deleteConfirmTag").attr("tagid");
        $.post(backendurl + "/delete", {id: tagid}).then(function(){
          $(".btn-modal-close").click();
          Socket.send("get all tags");
            $(".tag-label").css("border-left", "none");
            $(".tag-label").css("background-color", "white");
        });
      }
    }
  }

</script>

<style scoped>
  .add-tag {
    margin-bottom: 30px;
  }
  .tag-label {
    padding: 10px 0;
  }
  .tag-name-wrapper {
    margin-top: 7px;
  }
  .dot-img-wrapper {
    margin-top: 15px;
  }
  .dot-img {
    border-radius: 50%;
    float: right;
    width: 10px;
    height: 10px;
  }
</style>
