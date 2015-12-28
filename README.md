先看效果图： 
 ![示意图](http://upload-images.jianshu.io/upload_images/1216032-ce5dcef45df13309.gif?imageMogr2/auto-orient/strip)

这两天做了一个项目，需求如同置前标签，然后就对tagcloudeview稍做修改做了这么一个demo。不为别的，只为以后自己用的时候方便拷贝。
云标签开源地址https://github.com/kingideayou/TagCloudView
在源码里面加了两个方法
```
/**修改某些位置定点颜色**/

public void setTagsByPosition(HashMap<Integer, Boolean> positions, List<String> tagList){

this.tags = tagList;

this.removeAllViews();

if (tags != null && tags.size() > 0) {

for (int i = 0; i < tags.size(); i++) {

TextView tagView = (TextView) mInflater.inflate(mTagResId, null);

if (mTagResId == DEFAULT_TAG_RESID) {

tagView.setBackgroundResource(mBackground);

tagView.setTextSize(TypedValue.COMPLEX_UNIT_SP, mTagSize);

if (positions.get(i)){

tagView.setTextColor(mSeclectTagColor);

}else{

tagView.setTextColor(mTagColor);

}

}

LayoutParams layoutParams = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);

tagView.setLayoutParams(layoutParams);

tagView.setText(tags.get(i));

tagView.setTag(TYPE_TEXT_NORMAL);

final int finalI = i;

tagView.setOnClickListener(new OnClickListener() {

@Override

public void onClick(View v) {

if (onTagClickListener != null) {

onTagClickListener.onTagClick(finalI);

}

}

});

addView(tagView);

}

}

postInvalidate();

}

/**最前面的修改颜色**/

public void setTagsByLength(int length,List<String> tagList){

this.tags = tagList;

this.removeAllViews();

if (tags != null && tags.size() > 0) {

for (int i = 0; i < tags.size(); i++) {

TextView tagView = (TextView) mInflater.inflate(mTagResId, null);

if (mTagResId == DEFAULT_TAG_RESID) {

tagView.setBackgroundResource(mBackground);

tagView.setTextSize(TypedValue.COMPLEX_UNIT_SP, mTagSize);

if (i >= length){

tagView.setTextColor(mTagColor);

}else{

tagView.setTextColor(mSeclectTagColor);

}

}

LayoutParams layoutParams = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);

tagView.setLayoutParams(layoutParams);

tagView.setText(tags.get(i));

tagView.setTag(TYPE_TEXT_NORMAL);

final int finalI = i;

tagView.setOnClickListener(new OnClickListener() {

@Override

public void onClick(View v) {

if (onTagClickListener != null) {

onTagClickListener.onTagClick(finalI);

}

}

});

addView(tagView);

}

}

postInvalidate();

}
```

一目了然的方法，所以不多做解释 另外加了一个选中字体颜色的全局常量，和一个int变量
```
private static final int SELCECT_TEXT_COLOR = R.color.yellow_bg;//选中后的标签颜色 
private int mSeclectTagColor;
```
 在styles.xml中给TagCloudView增加了一个选中字体颜色的attr
```
<attr name="tcvSeclecTextColor" format="reference" />
```
剩下就是运用的地方不多说，直接上代码
```
public class MainActivity extends AppCompatActivity {

private TagCloudView normalTagView;//标准

private TagCloudView selectTagUseView;//置前

private TagCloudView positionsView;//定点

private List<String> AllTagsNormal = new ArrayList<>(0);//整个标签存放集合

private List<String> AllTagsSelect = new ArrayList<>(0);//整个标签存放集合

private List<String> AllTagsPosition = new ArrayList<>(0);//整个标签存放集合

private List<String> selectTags = new ArrayList<>(0);//选中的标签

private List<String> notSelectTags = new ArrayList<>(0);//未选中的标签

private HashMap<Integer, Boolean> map = new HashMap<>(0);//记录选择的位置

@Override

protected void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);

setContentView(R.layout.activity_main);

Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);

normalTagView = (TagCloudView) findViewById(R.id.normalTag);

selectTagUseView = (TagCloudView) findViewById(R.id.selcetTagUse);

positionsView = (TagCloudView) findViewById(R.id.positionsTag);

setSupportActionBar(toolbar);

for (int i = 0; i < 15; i++) {

AllTagsNormal.add("普通标签" + i);

AllTagsSelect.add("置前标签" + i);

AllTagsPosition.add("定点标签" + i);

map.put(i, false);

}

normalTagView.setOnTagClickListener(new TagCloudView.OnTagClickListener() {

@Override

public void onTagClick(int position) {

Snackbar.make(normalTagView, AllTagsNormal.get(position), Snackbar.LENGTH_LONG)

.setAction("Action", null).show();

}

});

selectTagUseView.setOnTagClickListener(new TagCloudView.OnTagClickListener() {

@Override

public void onTagClick(int position) {

if (selectTags.contains(AllTagsSelect.get(position))) {//如果选中的里面有 就删掉 扔到未选中的里面去

selectTags.remove(position);

notSelectTags.add(AllTagsSelect.get(position));

} else {

selectTags.add(AllTagsSelect.get(position));//

notSelectTags.remove(position - selectTags.size() + 1);

}

Snackbar.make(selectTagUseView, AllTagsSelect.get(position), Snackbar.LENGTH_LONG)

.setAction("Action", null).show();

AllTagsSelect.clear();//清空，重新装数据

AllTagsSelect.addAll(selectTags);

AllTagsSelect.addAll(notSelectTags);

bindSelectUseView(selectTags.size());

}

});

positionsView.setOnTagClickListener(new TagCloudView.OnTagClickListener() {

@Override

public void onTagClick(int position) {

bindPositionView(position);

Snackbar.make(positionsView, AllTagsPosition.get(position), Snackbar.LENGTH_LONG)

.setAction("Action", null).show();

}

});

normalTagView.setTags(AllTagsNormal);

int selectLength = 4;

bindSelectUseView(selectLength);

//用一个hashmap存放当前位置是否需要变色

bindPositionView(3);

bindPositionView(6);

bindPositionView(9);

}

/**

 * 定点标签记录和view变化

 **/

private void bindPositionView(int position) {

for (int i = 0; i < AllTagsPosition.size(); i++) {

if (i == position) {

if (map.get(i)) {

map.put(i, false);

} else {

map.put(i, true);

}

} else {

if (map.get(i)) {

map.put(i, true);

} else {

map.put(i, false);

}

}

}

positionsView.setTagsByPosition(map, AllTagsPosition);

for (int i = 0; i < AllTagsPosition.size(); i++) {

if (map.get(i)) {

positionsView.getChildAt(i).setBackgroundResource(R.drawable.edit_style_yellow);

}

}

}

/**

 * 选中标签的运用

 **/

private void bindSelectUseView(int selectLength) {

selectTagUseView.setTagsByLength(selectLength, AllTagsSelect);

selectTags.clear();

notSelectTags.clear();

for (int i = 0; i < AllTagsSelect.size(); i++) {

if (i < selectLength) {

selectTags.add(AllTagsSelect.get(i));//选中的存放入集合

selectTagUseView.getChildAt(i).setBackgroundResource(R.drawable.edit_style_yellow);

} else {

notSelectTags.add(AllTagsSelect.get(i));//未选中的存放入集合

}

}

}

@Override

public boolean onCreateOptionsMenu(Menu menu) {

// Inflate the menu; this adds items to the action bar if it is present.

getMenuInflater().inflate(R.menu.menu_main, menu);

return true;

}

@Override

public boolean onOptionsItemSelected(MenuItem item) {

// Handle action bar item clicks here. The action bar will

// automatically handle clicks on the Home/Up button, so long

// as you specify a parent activity in AndroidManifest.xml.

int id = item.getItemId();

//noinspection SimplifiableIfStatement

if (id == R.id.action_settings) {

return true;

}

return super.onOptionsItemSelected(item);

}

}
```
