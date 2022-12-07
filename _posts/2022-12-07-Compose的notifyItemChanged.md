---

layout: post
title: Compose的notifyItemChanged
subtitle: 
categories: android
tags: compose 技巧
---

## 前言

Compose中如果想监听列表的变化可以用mutableStateListOf来创建列表项，但是只能监听数据的增加/删除/替换，无法监听到数据内部的变化，即无法实现RV的notifyItemChanged。在数据变动时，除非变动的item移出屏幕外再移回来否则界面不会刷新。所以可以另辟蹊径，每次数据变动时copy一份数据，重新赋值，然后set回去，这样就实现了局部刷新。

Compose中常用分页库为Paging3，内部列表实现为ItemSnapshotList，继承的接口为AbstractList和List，无set方法，所以无法实现局部刷新

## 实现

实体类

```kotlin
data class TestEntity(
    val name: String,
    val url: String,
    val flag1: Boolean = true,
    val flag2: Boolean = true
)
```

ViewModel

```kotlin
class TestVM: ViewModel() {
    
    val testList = mutableStateListOf<TestEntity>()
    private var isLoadComplete = false
    
    val isRefresh = MutableStateFlow(false)
    
    private var page = 1
    
    fun getData(refresh: Boolean = true){
        if (refresh) {
            isLoadComplete = false
            page = 1
        } else {
            if (isLoadComplete) {
                return
            }
            page ++
        }
        isRefresh.tryEmit(true)
        viewModelScope.launch {
            if (refresh) {
                testList.clear()
            }
            val list = obtainData(page)
            testList.addAll(list)
            if (list.isEmpty()) {
                isLoadComplete = true
            }
            delay(1000)
            isRefresh.tryEmit(false)
        }
    }
    
    //这里改变数据，UI会随之刷新
    fun update(index: Int, entity: TestEntity){
        viewModelScope.launch {
            delay(1000)
            testList[index] = testList[index].copy(flag1 = entity.flag1, flag2 = entity.flag2)
        }
    }
    
    private fun obtainData(page: Int): List<TestEntity> {
        if (page > 3) {
            return emptyList()
        }
        return mutableListOf<TestEntity>().apply {
            repeat(5) { index ->
                add(
                    TestEntity(
                        name = "name $index",
                        url = "url $index",
                    )
                )
            }
        }
    }
}
```

```kotlin
@Composable
fun MainPage(){
    val vm = viewModel<TestVM>()
    val list = vm.testList
    val state by vm.isRefresh.collectAsState()
    //首次获取数据，因为key没有变化所以这里只会执行一次
    LaunchedEffect(key1 = Unit) {
        vm.getData()
    }
    //列表状态
    val listState = rememberLazyListState()
    val coroutineScope = rememberCoroutineScope()
    //判断列表是否滑动到底部
    //这里也可以写成在滑到距离底部几个的时候开始加载。举个例子👇
    // listState.firstVisibleItemIndex + listState.layoutInfo
    //  .visibleItemsInfo.size - listState.layoutInfo.totalItemsCount == 3
    //以上是在距离底部第三个的位置开始加载
    val isReachedBottom by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex + listState.layoutInfo
                .visibleItemsInfo.size == listState.layoutInfo.totalItemsCount
        }
    }
    //通过首个可见的下标来显示回到顶部的FloatActionButton
    //这里是当下标=6的时候显示
    val showBtn by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex > 5
        }
    }
    LaunchedEffect(Unit) {
        snapshotFlow { isReachedBottom }
            .collect {  isReached ->
                if (isReached) {
                    //获取更多数据
                    vm.getData(false)
                }
            }
    }
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(Color.LightGray)
    ) {
        SwipeRefresh(state = rememberSwipeRefreshState(isRefreshing = state), onRefresh = { vm.getData() }) {
            LazyColumn(
                state = listState
            ) {
                itemsIndexed(list) { index, item ->
                    Item(
                        reverseIndex = index,
                        entity = item,
                        click1 = {
                            vm.update(index, item.copy(flag1 = !item.flag1))
                        },
                        click2 = {
                            vm.update(index, item.copy(flag2 = !item.flag2))
                        }
                    )
                }
            }
        }
        AnimatedVisibility(
            visible = showBtn,
            enter = scaleIn(transformOrigin = TransformOrigin(0.5f, 1f)),
            exit = scaleOut(transformOrigin = TransformOrigin(0.5f, 1f))
        ) {
            Box(
                Modifier
                    .fillMaxSize()
                    .padding(bottom = 16.dp), contentAlignment = Alignment.BottomCenter
            ) {
                FloatingActionButton(onClick = {
                    coroutineScope.launch {
                        listState.animateScrollToItem(0)
                    }
                }) {
                    Icon(imageVector = Icons.Outlined.KeyboardArrowUp, contentDescription = "")
                }
            }
        }
    }
}

@Composable
fun Item(reverseIndex: Int, entity: TestEntity, click1: () -> Unit, click2: () -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .padding(12.dp, 4.dp)
            .clip(RoundedCornerShape(8.dp))
            .background(Color.White)
    ) {
        Text(text = "$reverseIndex")
        Divider()
        Text(
            text = entity.name,
            color = Color.Black,
            fontWeight = FontWeight.Bold,
            fontSize = 18.sp
        )
        Spacer(modifier = Modifier
                .fillMaxWidth()
                .height(8.dp))
        Text(
            text = entity.url,
            color = Color.LightGray
        )
        Row(
            modifier = Modifier
                .fillMaxWidth()
        ) {
            Button(
                onClick = { click1() },
                modifier = Modifier
                    .weight(1f)
                    .background(Color.Green)
            ) {
                Text(
                    text = "flag 1 ${entity.flag1}",
                    color = Color.White
                )
            }
            Button(
                onClick = { click2() },
                modifier = Modifier
                    .weight(1f)
                    .background(Color.Green)
            ) {
                Text(
                    text = "flag 2 ${entity.flag2}",
                    color = Color.White
                )
            }
        }
    }
}
```

## 吐槽

和原生的`RecyclerView`比起来真的是一个天上一个地下，卡是真特么的卡