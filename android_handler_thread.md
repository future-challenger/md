һ��`Handler`+`Thread`����һ�����ڴ���һ���̲߳����߳����е�ĳ���׶�
��ʱ��ʹ��Handler����UI��`AsyncTask`Ҳ�ǲ��һ���ô����ں�ִ̨��һ������
Ҳ������ĳһ�����ȵ�ʱ�����UI�����߻������ƣ��������Ǿ���`Thread`��`Handler`��ģ��һ��
`AsyncTask`��

```java
public class NormalActivity extends Activity {
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            // ...
        }
    };
}
```
��һ��Activity�������`NormalActivity`����ʼ��һ��`Handler`��Ȼ�������`Handler`
����д`handleMessage`�����������ﴦ���̷߳������ĸ���UI����Ϣ��Message msg����

�������ǰѷ����ĸ�����Ϣ���µ�һ��`TextView`�ϡ�û����ʾһ�����Ȱٷֱ�