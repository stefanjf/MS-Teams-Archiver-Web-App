<template>
  <v-container>
    <v-row class="text-center">
      <!-- <v-col cols="12">
        
      </v-col> -->

      <v-col class="">
     <v-alert
        border="bottom"
        color="grey darken-1"
        type="info"
        dark
        dense
      >
        <div class="mb-1 text--grey" style="text-align: left">Need assistance or to report a problem? Contact stefan.j.fernandez.mil@cvr.mil on CVR Teams or stefan.fernandez.1@us.af.mil.</div>

</v-alert>
        <v-expansion-panels class="pb-2">

           <v-expansion-panel class="grey lighten-2">
            <v-expansion-panel-header>
              <span><v-icon color="grey darken-2" left> mdi-help </v-icon> Frequently Asked Questions</span>
            </v-expansion-panel-header>
            <v-expansion-panel-content style="text-align: left">
                        <ul>
                <li class="mb-2">
                  <strong>Can I archive regular Chat groups? </strong></br>
                  <span >Unfortunately I have not found a way to access regular Chat groups. This workaround uses the Graph API token which does not have access to Chats. </span>
                </li>
                <li class="mb-2">
                  <strong>Can I migrate these Channel messages to AFNET/CHES Teams?</strong></br>
                  <span>No, that would require Teams admin access.</span>
                </li>
                <li class="mb-2">
                  <strong>What format should I choose?</strong></br>
                  <span>Probably HTML. If you want a Word document, use HTML and then copy/paste into Word. The JSON version is for anyone who wants all the data to parse themselves.</span>
                </li>
              </ul>
     
            </v-expansion-panel-content>
          </v-expansion-panel>

           <v-expansion-panel class="grey lighten-2">
            <v-expansion-panel-header>
              <span><v-icon color="grey darken-2" left> mdi-information-outline </v-icon> How to get an Access Token?</span>
            </v-expansion-panel-header>
            <v-expansion-panel-content style="text-align: left">
                     <ul>
                <li>
                  <span
                    >1. Navigate to
                    <a href="https://developer.microsoft.com/en-us/graph/graph-explorer"
                      >https://developer.microsoft.com/en-us/graph/graph-explorer</a
                    ></span
                  >
                </li>
                <li><span>2. Sign in with your CVR Teams account </span></li>
                <li><span>3. Click the 'Access Token' tab and copy the token. </span></li>
              </ul>
              </br>
              <img src="../../imgs/token.png" alt="ex" title="token" width="600" />
            </v-expansion-panel-content>
          </v-expansion-panel>
          
        </v-expansion-panels>

        <v-text-field v-model="accessToken" label="Enter your access token"></v-text-field>
        <v-btn color="green lighten-2" @click.stop="getListOfTeams">List Channels</v-btn>
      </v-col>

      <v-col class="mb-5" cols="12">
        <v-simple-table dense height="500px">
          <template v-slot:default>
            <thead>
              <tr>
                <th class="text-left">Team</th>
                <!-- <th class="text-left">id</th> -->
                <th class="text-left">Name</th>
                <th class="text-left">Download</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in myListOfChannels" :key="item.id">
                <td style="text-align: left">{{ item.teamName }}</td>
                <!-- <td>{{ item.id }}</td> -->
                <td style="text-align: left">{{ item.displayName }}</td>
                <td>
                  <v-btn small @click.stop="saveToHTMLFile(item)" class="mr-2">HTML</v-btn>
                  <v-btn small @click.stop="saveToJSONFile(item)">JSON</v-btn>
                </td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>

         <v-dialog
        v-model="initialWarning"
        width="500"
      >
        <v-card>
          <v-card-title class="headline grey lighten-2">
            Caution
          </v-card-title>
  
          <v-card-text class="mt-2">
            This is not an official DoD website, and you should always think twice before entering your access token into a random website.
            <br><br>
            Feel free to verify the source code here: <strong>https://github.com/stefanjf/CVR-Teams-Archiver-App</strong> 

            <br><br>
            Or reach out to me at <strong>stefan.fernandez.1@us.af.mil</strong> and I can verify that this website is non-malicious and secure.
          </v-card-text>

          <v-divider></v-divider>
  
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              text
              @click="clickPopup()"
            >
              I understand
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

               <v-dialog
        v-model="initialWarning2"
        width="500"
      >
        <v-card>
          <v-card-title class="headline grey lighten-2">
            FOUO/CUI
          </v-card-title>
  
          <v-card-text class="mt-2">
            If your channel contains FOUO/CUI information then only use this app from an appropriate computer system.
          </v-card-text>

          <v-divider></v-divider>
  
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              text
              @click="initialWarning2 = false"
            >
              I understand
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

        <v-dialog v-model="isLoading" hide-overlay persistent width="300">
          <v-card color="primary" dark>
            <v-card-text>
              {{ loadingText }}
              <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
            </v-card-text>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
var FileSaver = require("file-saver");

export default {
  name: "MessageArchiver",

  data: () => ({
    accessToken: "",
    myListOfChannels: [],
    allMessagesRaw: [],
    errors: "",
    isLoading: false,
    loadingText: "",
    initialWarning: true,
    initialWarning2: false,
  }),
  computed: {
    // a computed getter
    headers: function () {
      return { Authorization: this.accessToken };
    },
  },
  methods: {
    clickPopup: function () {
      this.initialWarning = false;
      this.initialWarning2 = true;
    },
    getListOfTeams: function () {
      this.isLoading = true;
      this.loadingText = "Getting your Teams and Channels";
      this.myListOfChannels = [];
      let url = "https://graph.microsoft.com/beta/me/joinedTeams";
      axios
        .get(url, { headers: this.headers })
        .then((response) => {
          // JSON responses are automatically parsed.
          for (const team of response.data["value"]) {
            axios
              .get("https://graph.microsoft.com/beta/teams/" + team["id"] + "/channels", {
                headers: this.headers,
              })
              .then((responseChannel) => {
                for (const channel of responseChannel.data["value"]) {
                  channel["teamName"] = team["displayName"];
                  channel["teamID"] = team["id"];
                  this.myListOfChannels.push(channel);
                }
                this.isLoading = false;
              })
              .catch((e) => {
                this.errors = e;
                this.isLoading = false;
              });
          }
        })
        .catch((e) => {
          this.errors = e;
          this.isLoading = false;
        });
    },
    async downloadRawMessages(item) {
      this.clearData()

      let linkToNextBatch = "";
      let url =
        "https://graph.microsoft.com/beta/teams/" + item["teamID"] + "/channels/" + item["id"] + "/messages?$top=100";
      let firstPull = await axios.get(url, { headers: this.headers });

      for (const message of firstPull.data["value"]) {
        this.allMessagesRaw.push(message);
      }

      if (this.isThereMoreMessages(firstPull)) {
        linkToNextBatch = firstPull.data["@odata.nextLink"];
        console.log("more messages found", firstPull);

        let nextPull;
        //Loop until no messages left
        while (true) {
          try {
            nextPull = await axios.get(linkToNextBatch, {
              headers: this.headers,
            });
          } catch (error) {
            console.log(error);
          }

          for (const message of nextPull.data["value"]) {
            this.allMessagesRaw.push(message);
          }

          if (this.isThereMoreMessages(nextPull)) {
            linkToNextBatch = nextPull.data["@odata.nextLink"];
          } else {
            break;
          }
        }
      }
      await this.addRepliesToRawMessages(item["teamID"], item["id"]);
    },
    async addRepliesToRawMessages(_groupID, _channelID) {
      console.log("start adding in replies");
      let linkToNextBatch = "";

      for (const msg of this.allMessagesRaw) {
        let url =
          "https://graph.microsoft.com/beta/teams/" +
          _groupID +
          "/channels/" +
          _channelID +
          "/messages/" +
          msg["id"] +
          "/replies";
        const firstPull = await axios.get(url, { headers: this.headers });

        msg["replies"] = [];
        for (const reply of firstPull.data["value"]) {
          msg["replies"].push(reply);
        }

        //Check for more
        if (this.isThereMoreMessages(firstPull)) {
          linkToNextBatch = firstPull.data["@odata.nextLink"];

          let nextPullReplies;
          //Loop until no messages left
          while (true) {
            //Get next pull of replies
            nextPullReplies = await axios.get(linkToNextBatch, {
              headers: this.headers,
            });

            //Add this batch to the main array of messages
            for (const replies2 of nextPullReplies.data["value"]) {
              msg["replies"].push(replies2);
            }

            //Check for more or break
            if (this.isThereMoreMessages(nextPullReplies)) {
              linkToNextBatch = nextPullReplies.data["@odata.nextLink"];
            } else {
              break;
            }
          }
        }
      }
    },
    isThereMoreMessages(response) {
      const check = typeof response.data["@odata.nextLink"];
      if (check != "undefined") {
        // console.log("more? yes", check);
        return true;
      } else {
        // console.log("more? no", check);
        return false;
      }
    },
    async saveToJSONFile(item) {
      this.isLoading = true;
      this.loadingText = "Downloading all messages. This may take a while...";

      await this.downloadRawMessages(item);

      this.isLoading = false;

      var blob = new Blob([JSON.stringify(this.allMessagesRaw)], {
        type: "text/plain;charset=utf-8",
      });
      const export_date = new Date();
      FileSaver.saveAs(blob, `${item.teamName}_${item.displayName}_archive_${export_date.toISOString()}.txt`);
    },
    async saveToHTMLFile(item) {
      this.isLoading = true;
      this.loadingText = "Downloading all messages. This may take a while...";

      await this.downloadRawMessages(item);
      let htmlString = "";

      //Sort root messages
      this.allMessagesRaw.sort(function compare(a, b) {
        var dateA = new Date(a.createdDateTime);
        var dateB = new Date(b.createdDateTime);
        return dateA - dateB;
      });

      for (const msg of this.allMessagesRaw) {
        let content = this.lodash.get(msg, "body.content", "unknown");
        const displayName = this.lodash.get(msg, "from.user.displayName", "unknown");
        const msgTime = this.lodash.get(msg, "createdDateTime", "unknown");
        const subject = this.lodash.get(msg, "subject", "") ? this.lodash.get(msg, "subject", "") + "<br>" : "";

        if (content) {
          content = content.replace(/\n+/g, "");
          content = content.replace(/\t+/g, "");
        }

        htmlString +=
          "<hr><hr><h3 style='margin-bottom: 3'>" +
          displayName +
          ":</h3><p style='margin-top: 0'>Sent: " +
          msgTime +
          "</p>" +
          "<b>" +
          subject +
          "</b>" +
          content +
          "<blockquote>";

        // Sort replies
        msg["replies"].sort(function compare(a, b) {
          var dateA = new Date(a.createdDateTime);
          var dateB = new Date(b.createdDateTime);
          return dateA - dateB;
        });

        for (const reply of msg["replies"]) {
          const user = this.lodash.get(reply, "from.user.displayName", "unknown");
          let replyContent = this.lodash.get(reply, "body.content", "unknown");
          const replyTime = this.lodash.get(reply, "createdDateTime", "unknown");

          if (replyContent) {
            replyContent = replyContent.replace(/\n+/g, "");
            replyContent = replyContent.replace(/\t+/g, "");
          }

          htmlString +=
            "<h3 style='margin-bottom: 3'>Reply From: " +
            user +
            "</h3>" +
            "<p style='margin-top: 0'>Sent: " +
            replyTime +
            "</p>" +
            replyContent;
        }
        htmlString += "</blockquote>";
      }

      this.isLoading = false;

      var blob = new Blob([htmlString], {
        type: "text/plain;charset=utf-8",
      });
      const export_date = new Date();
      FileSaver.saveAs(blob, `${item.teamName}_${item.displayName}_archive_${export_date.toISOString()}.html`);
    },
    clearData() {
      this.allMessagesRaw = []
    }
  },
};
</script>

