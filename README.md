在使用ListView的时候，我们通常会使用到其item的点击事件。而有些时候我们可能会用到item内部控件的点击操作，比如在item内部有个Button，当点击该Button时，删除所在的item。

效果图如下图所示

![itemdeleteclick](https://github.com/JZHowe/ListViewItemClick/blob/master/gif/ItemClick.gif)


----------


## 「Talk is cheap. Show me the code」 ##
怎么实现这个操作呢？先来看下代码：
先看布局文件activity_main.xml
只有一个ListView
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />

</LinearLayout>

```
再看ListView的item布局文件list_item.xml
包括了一个TextView和一个Button
其中Button添加了一个属性**android:focusable="false"** 
目的是为了不让Button强制获取item的焦点，否则item的点击事件就没用了

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="horizontal">

    <TextView
        android:id="@+id/item_tv"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center"
        android:text="this is text"
        android:textSize="24dp"/>

    <Button
        android:focusable="false"
        android:id="@+id/item_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="删除"/>
</LinearLayout>
```
自定义适配器MyAdapter.java
其中使用了接口回调，当点击删除按钮的时候，将点击item的position作为参数传出去
```
public class MyAdapter extends BaseAdapter {
    private Context mContext;
    private List<String> mList = new ArrayList<>();

    public MyAdapter(Context context, List<String> list) {
        mContext = context;
        mList = list;
    }

    @Override
    public int getCount() {
        return mList.size();
    }

    @Override
    public Object getItem(int i) {
        return mList.get(i);
    }

    @Override
    public long getItemId(int i) {
        return i;
    }

    @Override
    public View getView(final int i, View view, ViewGroup viewGroup) {
        ViewHolder viewHolder = null;
        if (view == null) {
            viewHolder = new ViewHolder();
            view = LayoutInflater.from(mContext).inflate(R.layout.list_item, null);
            viewHolder.mTextView = (TextView) view.findViewById(R.id.item_tv);
            viewHolder.mButton = (Button) view.findViewById(R.id.item_btn);
            view.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) view.getTag();
        }
        viewHolder.mTextView.setText(mList.get(i));
        viewHolder.mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mOnItemDeleteListener.onDeleteClick(i);
            }
        });
        return view;
    }

    /**
     * 删除按钮的监听接口
     */
    public interface onItemDeleteListener {
        void onDeleteClick(int i);
    }

    private onItemDeleteListener mOnItemDeleteListener;

    public void setOnItemDeleteClickListener(onItemDeleteListener mOnItemDeleteListener) {
        this.mOnItemDeleteListener = mOnItemDeleteListener;
    }

    class ViewHolder {
        TextView mTextView;
        Button mButton;
    }

}
```
最后是MainActivity.java
```
public class MainActivity extends AppCompatActivity {

    private ListView listView;
    private List<String> mList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initList();

        this.listView = (ListView) findViewById(R.id.listView);

        final MyAdapter adapter = new MyAdapter(MainActivity.this, mList);
        listView.setAdapter(adapter);
        //ListView item的点击事件
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                Toast.makeText(MainActivity.this, "Click item" + i, Toast.LENGTH_SHORT).show();
            }
        });
        //ListView item 中的删除按钮的点击事件
        adapter.setOnItemDeleteClickListener(new MyAdapter.onItemDeleteListener() {
            @Override
            public void onDeleteClick(int i) {
                mList.remove(i);
                adapter.notifyDataSetChanged();
            }
        });
    }

    /**
     * 初始化数据
     */
    private void initList() {
        for (int i = 0; i < 20; i++) {
            mList.add(i + "");
        }
    }
}
```
当点击删除按钮时，删除mList中的数据，并调用notifyDataSetChanged()方法刷新item数据，达到点击item内部删除按钮，以删除当前item的效果。
