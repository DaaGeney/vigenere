<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8 md6>
      <v-card min-width="1000px" max-width="1000px">
        <v-card-title class="headline">VIGENERE</v-card-title>
        <v-card-text>
          <!-- <p>Vuetify is a progressive Material Design component framework for Vue.js. It was designed to empower developers to create amazing applications.</p> -->
          <v-container fluid>
            <v-textarea
              clearable
              clear-icon="cancel"
              label="Mensaje encriptado"
              v-model="info.CipherText"
            ></v-textarea>
          </v-container>        
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn color="primary" @click="all">Continue</v-btn>
        </v-card-actions>     
      <v-card>
     
      <v-tabs vertical>
        <v-tab>
          <v-icon left>mdi-account</v-icon>
          Texto Cifrado
        </v-tab>
        <v-tab>
          <v-icon left>mdi-folder-key</v-icon>
          Key length
        </v-tab>
        <v-tab>
          <v-icon left>mdi-clipboard-text-play-outline</v-icon>
          Sub textos
        </v-tab>
        <v-tab>
          <v-icon left>mdi-key</v-icon>
          Llave sugerida
        </v-tab>
        <v-tab>
          <v-icon left>mdi-text</v-icon>
          Texto 
        </v-tab>
        <v-tab-item>
          <v-card flat>
            <v-card-text>
              <v-textarea
           
              v-model="cipher"
              readonly
            ></v-textarea>
                <!-- {{info.CipherText}} -->
            </v-card-text>
          </v-card>
        </v-tab-item>
        <v-tab-item>
          <v-card flat>
            <v-card-text class="headline">
              <p >
                {{info.KeyLength}}
              </p>
            </v-card-text>
          </v-card>
        </v-tab-item>
        <v-tab-item>
          <v-card flat>
            <v-card-text>
              <v-list-item
            v-for="i in info.Subtexts"
            :key="i"
            link
          > <v-list-item-title v-text="i"></v-list-item-title></v-list-item>
          
            </v-card-text>
          </v-card>
        </v-tab-item>
         <v-tab-item>
          <v-card flat>
            <v-card-text>
            <p class="headline" >
                {{info.GuessedKey}}
              </p>
            </v-card-text>
          </v-card>
        </v-tab-item>
         <v-tab-item>
          <v-card flat>
            <v-card-text>
              <v-textarea
              v-model="info.PlainText"
              readonly
            ></v-textarea>
              
            </v-card-text>
          </v-card>
        </v-tab-item>
      </v-tabs>
    </v-card>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
export default {
  data() {
    return {
      info: {
        CipherText: "",
        KeyLength: 0,
        Subtexts: [],
        GuessedKey: "",
        PlainText: ""
      },
      cont:3,
      cipher:"",
      allRight:false,
      differencePositions: [],
      relativeFrequency: [],
      newFrequency: [],
      alphabet: ["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"]
    };
  },
  methods: {
    all: function(){
      this.cont=3
      this.descifrar()
    },
    descifrar: function() {
      if(this.info.CipherText==""){
        alert("Debes digitar un texto cifrado")
      }if(this.info.CipherText.length<=11){
        alert("Mensaje muy corto, digite mensaje mas largo para que el metodo funcione")
      } 
      else{
      this.allRight=true
      this.differencePositions= []
      this.relativeFrequency= []
      this.newFrequency=[]
      this.info.GuessedKey=""
      this.info.PlainText= ""
      this.info.Subtexts=[],
      this.info.KeyLength=0
      for (let i = 0; i < this.info.CipherText.length - 2; i++) {
        var temp =  this.info.CipherText[i] 
        for(let j = 1; j<this.cont;j++){
          temp+=this.info.CipherText[i + j]
        }
        
        var regex = new RegExp(temp, "ig"),
          result,
          indices = [];
        while ((result = regex.exec(this.info.CipherText))) {
          indices.push(result.index);
        }
        if (indices.length > 1) {
          for (let j = 1; j < indices.length; j++) {
            this.differencePositions.push(indices[j] - indices[j - 1]);
          }
        }
        indices = [];
        } 
      
      Array.prototype.unique = (function(a) {
        return function() {
          return this.filter(a);
        };
      })(function(a, b, c) {
        return c.indexOf(a, b + 1) < 0;
      });
      this.differencePositions = this.differencePositions.unique();
      this.calculate_mcd();
      }
    },
    calculate_mcd: function() {
      Math.gcd = function() {
        if (arguments.length == 2) {
          if (arguments[1] == 0) return arguments[0];
          else return Math.gcd(arguments[1], arguments[0] % arguments[1]);
        } else if (arguments.length > 2) {
          var result = Math.gcd(arguments[0], arguments[1]);
          for (var i = 2; i < arguments.length; i++)
            result = Math.gcd(result, arguments[i]);
          return result;
        }
      };
      let mcd =
        this.differencePositions[0]
      for (let i = 1; i < this.differencePositions.length; i++) {
        mcd = Math.gcd(mcd, this.differencePositions[i]);
      }
      this.info.KeyLength = mcd;
      this.cont++
      this.info.KeyLength>1 ? this.calculateSubText(): this.descifrar()
      // this.calculateSubText()
    },
    calculateSubText: function() {
      for (let i = 0; i < this.info.KeyLength; i++) {
        let temp = "";
        for (
          let j = i;
          j < this.info.CipherText.length;
          j += this.info.KeyLength
        ) {
          temp += this.info.CipherText[j];
        }
        this.info.Subtexts.push(temp);
      }

      this.calculateFrequency();
    },
    calculateFrequency: function() {
      let count = 0;
      let aux = [];
      this.relativeFrequency= [];
      this.info.Subtexts.forEach(cadena => {
        this.alphabet.forEach(caracter => {
          count = 0;
          for (let i = 0; i < cadena.length; i++) {
            if (cadena[i].toLowerCase() === caracter.toLowerCase()) count++;
          }
          aux.push(count);
        });
        this.relativeFrequency.push(aux);
        aux = [];
      });

      this.getKey();
    },
    getKey: function() {
      this.info.GuessedKey=""
      this.relativeFrequency.forEach(listFrequency => {
        let aux = [];
        for (let i = 0; i < listFrequency.length; i++) {
          let indice1 = 0,
            indice2 = 0;
          indice1 = i + 4;
          indice2 = indice1 + 11;
          if (indice1 >= listFrequency.length)
            indice1 = indice1 - listFrequency.length;
          if (indice2 >= listFrequency.length)
            indice2 = indice2 - listFrequency.length;
          aux.push(
            listFrequency[i] + listFrequency[indice1] + listFrequency[indice2]
          );
        }
        this.newFrequency.push(aux)
        ;
      });
      console.log(this.newFrequency)
      let key = "",
        top = 0,
        index;
      this.newFrequency.forEach(list => {
        top = 0;
        list.forEach(element => {
          if (element > top) (top = element), (index = list.indexOf(element));
        });
        key += this.alphabet[index];
      });
      this.info.GuessedKey = key;
      let keyRepeat =""
      for (let i = 0; i < this.info.CipherText.length; i+=this.info.GuessedKey.length) {
        keyRepeat+=this.info.GuessedKey
      }
      for (let i = 0; i < this.info.CipherText.length; i++) {
        this.info.PlainText+=this.vigenereDecryption(keyRepeat[i],this.info.CipherText[i])
      }
      console.log(this.info);
      this.cipher=this.info.CipherText
    },
    vigenereDecryption(valueKey, valueCipher) {
      let values=[["A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"],
                  ["B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A"],
                  ["C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B"],
                  ["D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C"],
                  ["E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D"],
                  ["F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E"],
                  ["G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F"],
                  ["H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G"],
                  ["I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H"],
                  ["J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I"],
                  ["K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J"],
                  ["L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K"],
                  ["M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L"],
                  ["N","O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M"],
                  ["O","P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N"],
                  ["P","Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O"],
                  ["Q","R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P"],
                  ["R","S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q"],
                  ["S","T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R"],
                  ["T","U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S"],
                  ["U","V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T"],
                  ["V","W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U"],
                  ["W","X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V"],
                  ["X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W"],
                  ["X","Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W"],
                  ["Y","Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X"],
                  ["Z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y"],

]
      let aux = values[this.alphabet.indexOf(valueKey)], aux2 = aux.indexOf(valueCipher)
      return this.alphabet[aux2]
    }
  }
};
</script>
