# irods_py_rule_pretty_printer

Instructions:
   1. Clone this repository
      ```
      SOME_DIR=~/github
      mkdir -p ${SOME_DIR}
      cd ${SOME_DIR}/irods_py_rule_pretty_printer
      ```
   1. As user `irods` make a symbolic link in `/etc/irods` to the myinspect module
      ```
      sudo su - irods -c "ln -s ${SOME_DIR}/myinspect.py /etc/irods"
      ```
   1. create a PEP (Policy Enforcement Point) within `/etc/irods/core.py` ; as an example, prepend following text to that file:
      ```
      import myinspect, cStringIO
      import irods_types

      def r_of_s (o):
          return repr(str(o))

      irods_types_callback = (

        [ irods_types.c_string_array, lambda obj: [ r_of_s( i ) for i in obj ]
        ],

        [ irods_types.char_array, lambda obj: (r_of_s(obj),)
        ],

        [ irods_types.KeyValPair , lambda obj: ( ( str( obj.key[i] ), str( obj.value[i] ) )  for i in range(obj.len) )
      ],
      )

      def pep_api_data_obj_copy_post(rule_args, callback, rei):
          s = cStringIO.StringIO()
          myinspect.myInspect( rule_args, stream = s, types_callback = irods_types_callback )
          callback.writeLine("serverLog","pep_api_data_obj_copy_post")
          callback.writeLine("serverLog","s -> \n\t{}\n".format(s.getvalue()))
      ```
   1. cause the PEP to fire:
   ```
   imkdir srcDir ; touch VERSION.json ; iput VERSION.json srcdir
   imkdir dstDir
   icp srcDir/VERSION.json dstDir

   ```
---

Now examine the tail end of the most recent ``~irods/log/rodsLog.XXXX.XX.XX`

```
Jul 22 16:25:27 pid:6816 NOTICE: writeLine: inString = pep_api_data_obj_copy_post
Jul 22 16:25:27 pid:6816 NOTICE: writeLine: inString = s ->
        &<type 'list'> @ 0x7f11bdf3fe18
*LIST(4)
~0
    &<type 'str'> @ 0x7f11bdf33340
    *OBJECT of class "str":
    ='api_instance'
~1
    &<class 'irods_types.PluginContext'> @ 0x7f11bc66e050
    *OBJECT of class "PluginContext":
    >rule_results ->
        &<type 'str'> @ 0x7f11c74b3508
        *OBJECT of class "str":
        =''
~2
    &<class 'irods_types.DataObjCopyInp'> @ 0x11b82f0
    *OBJECT of class "DataObjCopyInp":
    >destDataObjInp ->
        &<class 'irods_types.DataObjInp'> @ 0x7f11c741ba60
        *OBJECT of class "DataObjInp":
        >condInput ->
            &<class 'irods_types.KeyValPair'> @ 0x7f11c741bad0
            *OBJECT of class "KeyValPair":
        >createMode ->
            &<type 'int'> @ 0xfb7e30
            *OBJECT of class "int":
            =0
        >dataSize ->
            &<type 'long'> @ 0x7f11c7410170
            *OBJECT of class "long":
            =224L
        >numThreads ->
            &<type 'int'> @ 0xfb7e30 (seen)
            =0
        >objPath ->
            &<class 'irods_types.char_array'> @ 0x7f11bc66baa0
            *OBJECT of class "char_array":
            |'/tempZone/home/rods/dstDir/VERSION.json'
        >offset ->
            &<type 'long'> @ 0x7f11be240108
            *OBJECT of class "long":
            =0L
        >openFlags ->
            &<type 'int'> @ 0xfb7e18
            *OBJECT of class "int":
            =1
        >oprType ->
            &<type 'int'> @ 0xfb7d58
            *OBJECT of class "int":
            =9
        >specColl ->
            &<type 'NoneType'> @ 0x7f11bec59110
            *OBJECT of class "NoneType":
            =None
    >srcDataObjInp ->
        &<class 'irods_types.DataObjInp'> @ 0x7f11c741bb40
        *OBJECT of class "DataObjInp":
        >condInput ->
            &<class 'irods_types.KeyValPair'> @ 0x7f11c741bbb0
            *OBJECT of class "KeyValPair":
            |('translatedPath', '')
            |('phyOpenBySize', '')
            |('resc_hier', 'demoResc')
        >createMode ->
            &<type 'int'> @ 0xfb7e30 (seen)
            =0
        >dataSize ->
            &<type 'long'> @ 0x7f11c7410190
            *OBJECT of class "long":
            =-1L
        >numThreads ->
            &<type 'int'> @ 0xfb7e30 (seen)
            =0
        >objPath ->
            &<class 'irods_types.char_array'> @ 0x7f11bc66bb90
            *OBJECT of class "char_array":
            |'/tempZone/home/rods/srcdir/VERSION.json'
        >offset ->
            &<type 'long'> @ 0x7f11be240150
            *OBJECT of class "long":
            =0L
        >openFlags ->
            &<type 'int'> @ 0xfb7e30 (seen)
            =0
        >oprType ->
            &<type 'int'> @ 0xfb7d40
            *OBJECT of class "int":
            =10
        >specColl ->
            &<type 'NoneType'> @ 0x7f11bec59110
            *OBJECT of class "NoneType":
            =None
~3
    &<class 'irods_types.TransferStat'> @ 0x7f11bc66b5f0
    *OBJECT of class "TransferStat":
    >len ->
        &<type 'int'> @ 0xfb7e30 (seen)
        =0
    >offset ->
        &<type 'long'> @ 0x7f11c74101b0
        *OBJECT of class "long":
        =224L
---

```
