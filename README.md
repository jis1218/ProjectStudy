##### RecyclerView에 item list view를 add해줄 때 화면에 꽉차게 나오지 않았다
##### 해결을 다음과 같이 해주면 된다.
```java
@Override
    public MyHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.frag_myinfo, null);
```
##### 에서
```java
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.frag_myinfo, parent, false);
        return new MyHolder(view);
    }
```
##### 로 바꿔주면 된다.

##### CoordinatorLayout에서 배운점

##### 1. 앱바의 크기 설정은 value에 dimen을 하나 만들어서 사용한다.
```
<resources>
        <dimen name="detail_backdrop_height">256dp</dimen>
        <dimen name="card_margin">16dp</dimen>
        <dimen name="fab_margin">16dp</dimen>
        <dimen name="list_item_avatar_size">100dp</dimen>
</resources>
```
##### 이렇게 하면 스크롤을 위로 올릴 때 줄어들게 된다.

##### 2. 앱바 안에 Layout 넣어 함께 줄어들게 하기
##### CollapsingToolbarLayout에 LinearLayout을 넣는다.
```
<android.support.design.widget.CollapsingToolbarLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="@color/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
```

##### CollapsingToolbarLayout의 scrollFlags에 설정을 해놓으면 그 밑에 있는 view도 영향을 받는 듯 하다.

##### 3. Scroll에 상관없이 특정 뷰를 특정 위치에 계속 놔두기
```
<android.support.design.widget.FloatingActionButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|right"
    android:layout_margin="16dp"
    android:src="@mipmap/ic_launcher"
    app:layout_anchor="@id/myinfoRecyclerView"
    app:layout_anchorGravity="bottom|right|end"/>
```
##### layout_anchor에 위치할 뷰와 gravitiy를 설정해두면 된다.
##### appbar에 놔두면 appbar가 사라지는 것을 볼 수 있다.
```
<android.support.design.widget.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_launcher_background"
        app:layout_anchor="@id/appbar"
        app:layout_anchorGravity="bottom|end"/>
```

##### 4. CoordinatorLayout을 쓸 때는 AppBarLayout쓰는 것이 default인 듯 하다
##### 참고 : AppBarLayout currently expects to be the direct child nested within a CoordinatorLayout according to the official [Google docs]

##### 5. RecyclerView(scorll 할 view)에는 app:layout_behavior 속성을 설정해놓는데 아래 글을 참고하면 된다.
##### Next, we need to define an association between the AppBarLayout and the View that will be scrolled. Add an app:layout_behavior to a RecyclerView or any other View capable of nested scrolling such as NestedScrollView. The support library contains a special string resource @string/appbar_scrolling_view_behavior that maps to AppBarLayout.ScrollingViewBehavior, which is used to notify the AppBarLayout when scroll events occur on this particular view. The behavior must be established on the view that triggers the event.

##### 6. 계속 view를 새롭게 하게 되면 Behavior가 기존 뷰, 새로운 뷰 중 어느 것을 참고해야 할지 몰라 Behavior가 작동하지 않게 된다.
```java
frameLayout.addView(new MyInfoLayout(context));
```

##### 아래 코드를 추가해줘야 한다.
```java
public boolean onNavigationItemSelected(@NonNull MenuItem item) {
        int id = item.getItemId();
        if(current_id == id)
            return false;

        frameLayout.removeAllViews();
        current_id = id;
        switch (id) {
            case R.id.navigation_newspeed:
                // View 가 이미 있는지 체크
                frameLayout.addView(new NewsPeedView(context));

                /*
                // 좀 더 쉽게 표현하면
                ((AppCompatActivity)context).getSupportFragmentManager()
                        .beginTransaction()
                        .add(R.id.frameLayout, new NewsPeedFragment())
                        .commit();
                */

                return true;
            case R.id.navigation_search:
                return true;
            case R.id.navigation_post:
                return true;
            case R.id.navigation_notification:
                return true;
            case R.id.navigation_profile:
                frameLayout.addView(new MyInfoLayout(context));
                return true;
        }
        return false;
    }
```

##### 7. SQLite를 이용해 검색기록 남기기

##### SQLite DB 생성 후 DB에 있는 내용을 ArrayList에 담는다. 담을 때는 list.add(0, data)로 하여 recyclerView에서 가장 나중에 add된 data가 먼저 보여지도록 한다.
```java
Cursor cursor = connection.rawQuery(query, null);
        while(cursor.moveToNext()){
            String word = cursor.getString(1);
            list.add(0, word);
        }
```
##### 중복되는 데이터는 insert 하기 전에 중복 데이터를 지워주는 query를 실행하면 된다.
```java
private void insert(){
        String word = editSearch.getText().toString();
        HistoryDAO dao = new HistoryDAO(getContext());
        String deleteQuery = "delete from history where word = '" + word + "'";
        String insertquery = "insert into history(word)" + " values('" + word + "')";
        dao.readQuery(deleteQuery);
        dao.readQuery(insertquery);
        readList();
    }
private void readList(){
        searchRecyclerAdapter.notifier(dao.read());
    }
```

##### 8. 한 RecyclerView에서 여러개의 ViewHolder를 이용하고 싶을 때 어떻게 해야 할까??

##### 링크 참고 - > https://stackoverflow.com/questions/25914003/recyclerview-and-handling-different-type-of-row-inflation

##### > Holder를 여러개 만들었으면 itemType에 따라 다음과 같이 해주면 된다.
```java
@Override
    public void onBindViewHolder(ViewHolder holder, int position) {

        final int itemType = getItemViewType(position);

        if (itemType == ITEM_TYPE_NORMAL) {
            ((MyNormalViewHolder)holder).bindData((MyModel)myData[position]);
        } else if (itemType == ITEM_TYPE_HEADER) {
            ((MyHeaderViewHolder)holder).setHeaderText((String)myData[position]);
        }
    }
```

##### 9. Retrofit과 Observable을 사용하여 Post를 날리고 그 결과값을 받으려고 하는데 그것이 잘 안됨
```java
DataService dataService = getDataFromDB().create(DataService.class);
        Observable<Response<SearchResponse>> observable = dataService.observable(word);
        observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(data->{
                }
```

##### 다음과 같은 에러가 뜸
```
java.lang.IllegalStateException: Expected BEGIN_OBJECT but was BEGIN_ARRAY at line 1 column 2 path $
```

##### 이유는 나는 Object 형태로 값을 받으려고 하지만 실제 내가 받는 데이터는 Array라는 뜻임
##### 아래 링크 참조
##### https://stackoverflow.com/questions/24154917/retrofit-expected-begin-object-but-was-begin-array

##### 다음과 같이 바꾸니 동작함
```java
Observable<Response<SearchResponse>> observable -> Observable<ArrayList<Response<SearchResponse>>> observable
```

##### 10. TextWatcher를 RxBinding으로 고침 -> 코드 줄 수가 많이 줄어듬
```java
TextWatcher textWatcher = new TextWatcher() {
           @Override
           public void beforeTextChanged(CharSequence s, int start, int count, int after) {

           }

           @Override
           public void onTextChanged(CharSequence s, int start, int before, int count) {
               Log.d("확인", "onTextChanged: " + s.toString() + start + before + count);
               if(count>0){
                  recyclerSearchHistory.setAdapter(searchRecyclerResultAdapter);
               }else if(start==0){
                   recyclerSearchHistory.setAdapter(searchRecyclerAdapter);
               }
           }

           @Override
           public void afterTextChanged(Editable s) {

           }
       };

       editSearch.addTextChangedListener(textWatcher);
```
##### 위 코드를 아래와 같이 바꿈
```
RxTextView.textChanges(editSearch)
                .subscribe(ch ->{
                    if(ch.length()>0){
                        recyclerSearchHistory.setAdapter(searchRecyclerResultAdapter);
                    }else{
                        recyclerSearchHistory.setAdapter(searchRecyclerAdapter);
                    }
                });

```

##### 11.
```java
public <T> T readObjectData(...
        ^  ^
        |  + Return type
        + Generic type argument
```

##### 12 PUT과 PATCH의 차이점
##### PUT은 전체 데이터를 다 보내주지만 PATCH는 변경된 데이터만 보낼 수 있다.
##### PATCH와 Multipart로 데이터 보내는 방법
```java
    @Multipart
    @PATCH("/member/userprofile/update/")
    Observable<Response<UserEditProfile>> userEditProfile(@Header("Authorization") String token, @Part MultipartBody.Part filePart, @Part("username") RequestBody username);
```
//이미지를 선택했을 경우 filepart에 이미지를 넣어준다.
        File file = new File(photoList.get(0).getImagePath());
        MultipartBody.Part filePart = MultipartBody.Part
        .createFormData("img_profile", file.getName(), RequestBody.create(MediaType.parse("image/*"), file));

        RequestBody username = RequestBody.create(MediaType.parse("text/plain"), editNameEditProfile.getText().toString());
```
##### 13. 데이터와 서버의 동기화
##### https://stackoverflow.com/questions/10829371/sync-data-between-android-app-and-webserver (2012년)
##### 안드로이드에서는 sync adapter를 이용하라고 하네

##### 14. 앱이 켜져있을 때, 꺼져있을 때 Noti 방식이 다름
##### http://trandent.com/board/Android/detail/744
##### 앱이 꺼져있을 때는 sendPushNotification이 작동하지 않는다. 이럴 땐 fcm api를 직접 호출하면 된다는데 무슨 얘긴지 모르겠다

##### 15. 앞의 activity에서 바뀐 정보를 뒤에 activity에 적용하기
##### onResume()에 바뀌는 코드를 넣어줬다.

##### 16. 한 Activity에서 다른 Activity로 각각의 다른 intent에 담긴 정보를 받아야 한다면
intent.setAction(), intent.getAction()을 이용하자!

##### 17. Presenter에서 다음과 같이 코드를 짰다.
```java
@Override
    public void setPresenter(PostContract.iPresenter presenter) {
        this.presenter = presenter;
        this.presenter.loadPostContent(cover);
        this.presenter.loadReply(cover);
    }
```
병렬적으로 실행되는 것이 loadPostContent안에서 loadReply를 실행하는 것보다 훨씬 효율적이고 안정적이다.
```java
Observable<Response<PostContentResult>> observable = repository.getPostContentList(postPk);
        observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(data -> {
                            if (data.isSuccessful()) {
                                boolean checkIfFollowing = false;
                                Log.e(TAG, "loadPostContent: 데이터 로드 완료");
                                view.hideProgress();
                                //list = loadReply(cover); 이렇게 안에서 실행되면 늦는다. 되도록이면 네트워크 연결 코드 안에 네트워크 연결 코드를 넣지 말자
```



##### 6. CollapsingToolbarLayout에 이 설정을 꼭 해줘야 사라지는 효과가 난다
```
app:contentScrim="@color/colorPrimary"
```


##### SurfaceView
##### 그림을 그릴 때 그려지면 바로 찍는다.

##### Multipart로 보내는 방법
##### 1. PostMan을 꼭꼭 활용하자!!!!
```java
requestBodyMap.put("img_profile\"; filename=\""+file.getName(), requestFile);
```
