# IotSerialLib
Android屏端模拟Wifi模块与MCU通讯

该sdk实现Android屏端模拟wifi模组发送iot命令到mcu的功能，和经典的屏端到mcu通过字节格式发送串口数据不一样，该方法使mcu对接Android屏和普通的led屏一样，不用做两套代码，冰箱屏端串口协议统一。


引入：
implementation 'com.viomi.iotserial:iot_serial_lib:1.0.1'

1、配置数据上报handler


        SerialConvert.getInstance().setHandler(new CmdHandler() {
            @Override
            public void props(Map prop) {
                //屬性上报
                Iterator<Map.Entry<String,Object>> iterator=prop.entrySet().iterator();
                while (iterator.hasNext()){
                    Map.Entry<String,Object> entry=iterator.next();
                    logUtil.d(TAG,"prop，name="+entry.getKey()+",vulae"+entry.getValue());
                }

            }

            @Override
            public void event(EventPack pack) {
                //事件上报
                logUtil.d(TAG,"event，name="+pack.name);
                Iterator<Object> iterator= pack.propList.iterator();
                while (iterator.hasNext()){
                    logUtil.d(TAG,"event params，prop="+iterator.next());
                }
            }

            @Override
            public void model(String model) {
                logUtil.d(TAG,"model="+model);
            }

            @Override
            public void mcu_version(String version) {
                logUtil.d(TAG,"mcu_version="+version);
            }
        });
        
        
       2、打开/关闭该库的调试log打印
         SerialConvert.getInstance().enableLog(true);//开启log打印
         
       3、初始化
         SerialConvert.getInstance().open(this,IotType.PROTOCOL_TYPE_MIOT);
         PROTOCOL_TYPE_MIOT表示是miot  profile协议，暂时只支持这种。
         
      4、获取设备属性
         String[] props=new String[]{"Mode","RCSetTemp"};
        SerialConvert.getInstance().action("get_prop", props, new ConmonCallback<String>() {
            @Override
            public void onReceiveResult(@NonNull String result) {

                try {
                    JSONObject jsonObject=new JSONObject(result);
                    int code=jsonObject.getInt("code");
                    if(code==0){
                       JSONArray jsonArray= jsonObject.getJSONArray("result");
                        String mode= jsonArray.getString(0);
                        int   rcsettemp=jsonArray.getInt(1);
                        Log.i(TAG,"getprop,success,Mode="+mode+",RCSetTemp="+rcsettemp);
                    }else {
                        Log.i(TAG,"getprop fail,msg="+jsonObject.getString("msg"));
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        });
        
        5、action发送
           //设置冷藏室温度为4度
        Integer[] props=new Integer[]{4};
        SerialConvert.getInstance().action("setRCSetTemp", props, new ConmonCallback<String>() {
            @Override
            public void onReceiveResult(@NonNull String result) {

                try {
                    JSONObject jsonObject=new JSONObject(result);
                    int code=jsonObject.getInt("code");
                    if(code==0){
                        Log.i(TAG,"setRCSetTemp success ");
                    }else {
                        Log.i(TAG,"setRCSetTemp fail,msg="+jsonObject.getString("msg"));
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        });
        
       
        
        6、固件ota升级
             try {
            SerialConvert.getInstance().otaStart("/sdcard/mcu/ota.bin", new ProgressCallback() {
                @Override
                public void onResult(boolean isProcessing, int progress, String desc) {
                    Log.i(TAG,"mcuOta，isProcessing  ="+isProcessing+",progress="+progress);
                    if(isProcessing==false&&progress==100){
                        Log.i(TAG,"mcuOta success!");
                    }else if(progress<=0){
                        Log.i(TAG,"mcuOta fail,msg="+desc);
                    }
                }
            });
        } catch (SerialException e) {
            Log.i(TAG,"ota fail，msg ="+e.getMessage());
            e.printStackTrace();
            Toast.makeText(MainActivity.this, "ota 异常", Toast.LENGTH_SHORT).show();
        }
        
        7、关闭服务
        SerialConvert.getInstance().close();
