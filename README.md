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

##### 6. CollapsingToolbarLayout에 이 설정을 꼭 해줘야 사라지는 효과가 난다
```
app:contentScrim="@color/colorPrimary"
```


##### SurfaceView
##### 그림을 그릴 때 그려지면 바로 찍는다.
