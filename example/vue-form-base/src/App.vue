<style>

  /*
    Customize your VUE-FORM-BASE component using the following CSS-Classnames
    -------------------------------------------------------------------------
    

    INFORMATION: don't use '<style scoped>' for this definitions, because a 'scoped' definition is inside the component not accessable 
  
    
    // FORM-ID  
    #form-base  // Is the default name. If you need different CSS for two forms then change default value 'form-base'  
                   by setting a different 'id' <form-base id="custom-id" ... /> for each component    


    // GENERAL classes ----------------------------------------------------------------------------------          
    #form-base .collection,   // 
    #form-base .item,         // for example, give all items the same backgroundcolor
    #form-base .error,        // style here your error messages 

    // VALIDATE CSS Pseudoselectors
    #form-base .item input:invalid{ background-color: #fdd; }
    #form-base .item input:valid{ background-color: #dfd; }
    #form-base .item input:focus{ background-color: #ffd; }          
    

    // TYPE specific classes ----------------------------------------------------------------------------
    // most typenames are the same types of HTML5 <input> tag, like for example <input type="password" /> 
    // all type implementation are browser dependent and in some browsers different.
    // Use [text],  password, email, select, multiselect, list, file, radio, checkbox, number, range, url, date, time, week, month
    #form-base .item.text-type,
    #form-base .item.select-type,
    #form-base .item.multiselect-type,
    #form-base .item.checkbox-type,
    
    // make invalid css for multiselect type
    #form-base .item.multiselect-item input:invalid{ background-color: #fc5; }
    #form-base .item.multiselect-item select { height:6rem }


    // KEY specific classes -----------------------------------------------------------------------------
    // here you have direct access to each key in your data-object
    #form-base .item.lastname-key,
    #form-base .item.user-adress-city-key, // deep nested key

    // make invalid css for individual key
    #form-base .item.adress-city-key input:invalid{ background-color: #fda; }
    #form-base .item.lastname-key input:focus{ background-color: #ff9; }          
    
    
  */
    

  /* ------------set GENERAL CSS ------------*/
  #form-base .collection { padding: 12px}
  #form-base .collection .item{ background-color:#efe}
  /*#form-base .error { color:red}*/

  /* set input pseudoselectors valid, invalid, focus css for all items */
  #form-base .item input:invalid{ background-color: #fdd; }
  #form-base .item input:valid{ background-color: #dfd; }
  #form-base .item input:focus{ background-color: #ffd; }    


  /* ------------set TYPE CSS ------------*/

  /* set css for all items with type text */
  #form-base .text-type { color: #00d }

  
  /* ------------set KEY CSS ------------*/
 
  /* set individual invalid-css only for password key */
  #form-base .item.password-key input:invalid{ background-color: darkorange; color: white }
  
  /* set individual css for key mselect */
  #form-base .item.nested-selections-mselect-key select{ height:8rem; background-color: yellowgreen; color:white}
  #form-base .item.select-type select{ height:3rem; background-color: yellowgreen; color:white}
  
  /* set individual css for key fileselector */
  #form-base .item.fileselector-key .button { background-color: yellowgreen; color:white}
  #form-base .item.fileselector-key .path{ background-color: yellowgreen; color:white}
  
</style>


<template>

  <div id="app" class="container">
    <p>Not mutated Data:</p>
    <p>{{ data }}</p>
    
    <p>Reactive State:</p>    
    <p>{{ $store.state.data }}</p>
   
    <button class="btn" @click="reset()">Reset</button> 
       
      <div class="row">
      
          <div class="col s6">
            <!--use vue-form-base component-->
            <form-base :data="data" :schema="schema" data-state-name="data" schema-state-name="schema">
              <!--use named slots for each key -->       
              <div slot="nested-selections-mselect-key" class="card green lighten-4">This is a named Slot</div>       
            </form-base>
          </div>

          <div class="col s6">
            <form-base :data="data" :schema="schema" data-state-name="data" schema-state-name="schema">
            </form-base>
          </div>             
      
      </div>
  </div>
</template>


<script>
  import FormBase from './components/formBase.vue'
  import { cloneDeep, isString, isArray } from 'lodash'

  export default {

    data() {
      return {  
    
        data:{
          user: "smith",
          email:'smith@online.com',          
          password: '12345',
          nested:{
            checkbox: 'untouched',
            radio: 'Yoga', 
            color: '#bcdf2f',
            selections: {
              select: 'Resilience',
              mselect: ['Yoga', 'Resilience'],
            },
          },
          fileselector:'green-tea.jpg'

        },

        schema:{    
          // Schema Definition and available Properties
          /* 
            // recommended info https://www.wufoo.com/html5/

            schema:{ 
                order: number,          // controls order of displaying 
                
                type: string,           // ['text', 'list', 'password', 'email', 'url', 'select', 'multiselect', 'file', 'radio', 'checkbox', 'number', 'range', 'date', 'time', 'week', 'month'] 
                label: string,          // title of item    
                placeholder: string,    // placeholder text if value is empty 
                true: string,           // text if checkbox is checked  
                false: string,          // text if checkbox is unchecked 
                accept: string,         // only in type:'file' limit to accepted files -  audio/*, video/*, image/*, image/gif, .pdf, .doc  
                title: string,          // optional define your own validation message
                error: string,          // preset/set inline error msg
                css: string             // inject at item level one or more classes

                min: number,            // limit number or range
                max: number,            // limit number or range
                step: number,        
                maxlength: number,      // max length opf type text/email 
                
                pattern: regexString,

                multiple: bool,         // use with type:'file' select one or more files   
                required: bool, 
                disabled: bool, 
                readonly: bool, 
                hidden: bool,           // do not display item - could be dynamically set from another item

                options: array,         // used with type: radio, list, text, select, mselect 
                
                // this Function MUST return a value
                mapGet: function,       function( value, obj, state, schema ) { value, obj.key, obj.value, obj.schema ->  IMPORTANT! Always return incoming mapped value }
                
                // this Function MUST return a value or a promise
                mapSet: function,       function( value, obj, state, schema ) { value, obj.key, obj.value, obj.schema ->  IMPORTANT! Always return outgoing mapped value }
                
                // Validation 
                validate: true          // use inline error message
                validate: function,     function( validationMessage, obj, state, schema, validity ) { use validationMessage state to log or display validation message  }              
                noValidate: function,     function( value, obj, state, schema ) { use value, obj.key, obj.value, obj.schema to handle novalidation }
            }                    
          */ 
        
          user: { 
            type:'text', 
            label:`User (Try 'hide' to hide password, value is required)`,
            validate:true,
            required:true, 
            // mapGet: v => v.toUpperCase(),
            mapSet: (val, obj, data, schema) => { schema.password.hidden = val.toUpperCase() === 'HIDE'; return val}, 
          },

          password:{
            type:'password', 
             label:'Password (only [0-9], invalidate with individual CSS & logs error to console and to user field )', 
            required:true, 
            pattern:'[0-9]*',
            // validate undefined   // no validation 
            // validate:true,       // show inline error message          
            // validate:(msg) => console.warn(msg), 
            validate:(msg, obj, data, schema, validity) => { console.warn(msg); obj.schema.error = `CUSTOM ERROR: ${msg}`; schema.user.error ="Wrong Password for this User" }, 
            noValidate:(val, obj, data, schema) => { schema.user.error = null; obj.schema.error = null  }, 
          },        

          email: {type:'email',label:'Email ( injected css ):', validate:true, css: 'yellow lighten-4' }, 
      
          nested:{
            checkbox: { type:'checkbox', true:'Yes!', false:'Oh No!'},
            radio: { type:'radio', hidden:false, options:['Resilience','Green Tea','Yoga','Curry']}, 
            color:{ type:'color'}, 
            selections: {
              select: { type:'select', label:'Select One', options:['Resilience','Green Tea','Yoga','Curry']},
              mselect: { type:'multiselect', label:'Select Some', required:true,  validate:true, options:['Resilience','Green Tea','Yoga','Curry']},
            }, 
          },

          fileselector:{ type:'file', multiple:true, label:'Get Teas!' }  

        },

      }
    },

    methods:{

      reset(){
        this.$store.state.data = cloneDeep(this.data); 
        this.$store.state.schema = cloneDeep(this.schema)
      }  

    },

    components: { FormBase },

  }
</script>

