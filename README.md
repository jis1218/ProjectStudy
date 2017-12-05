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
