# 单元测试

## 目的
>> ### 检测代码逻辑是否正确
>>> ###### 代码覆盖率
>>> ###### 有效的测试用例(输入和输出能匹配)
>> ### 在不依赖外部条件的情况下运行
>>> ###### 外部代码
>>> ###### 外部进程

## 目标
>>> ###### 函数

## Input/Output/Exception
	

## 桩对象(Stub)
	模拟测试单元的外部依赖

## Assert
	预测结果

# Mockito

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ExternalSystemServiceImpl.class})
public class TestCustProfileManageServiceImpl {
    @InjectMocks
    private CustProfileManageServiceImpl custProfileManageService;

    @Mock
    private CustomerAggregateBean customerAggregateBean;

    @Spy
    @Autowired
    private ExternalSystemServiceImpl externalSystemServiceImpl;
}
```

### \@InjectMocks
	单元测试的目标,桩对象的注入目标

### \@Mock
+	纯模拟的桩对象

+	默认返回null

e.g.
```
Mockito.when(customerAggregateBean.getProfileBean(Mockito.argThat(new IsNot<>(new IsNull<>())))).thenReturn(customerProfileBean);
```

+	参数匹配
```
Mockito.argThat(Matcher<T> matcher);
Mockito.argThat(ArgumentMatcher<T> matcher);
```

+	模拟结果
> 固定返回
```
Mockito.when(customerAggregateBean.getProfileBean("a").thenReturn(null);
```
> 动态返回
```
Mockito.when(customerAggregateBean.getGrCardBean(Mockito.argThat(new IsNot<>(new IsNull<>())))).thenAnswer(invocation -> {
    Object args = invocation.getArguments();
    return null;
});
```

+	模拟异常
```
Mockito.doThrow(Class<? extends Throwable> toBeThrown).when(customerAggregateBean.getGrCardBean(Mockito.argThat(new IsNot<>(new IsNull<>()))));
```

### \@Spy
+	部分模拟的桩对象
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ExternalSystemServiceImpl.class})
public class TestCustProfileManageServiceImpl {
    @InjectMocks
    private CustProfileManageServiceImpl custProfileManageService;

    @Spy
    @Autowired
    private ExternalSystemServiceImpl externalSystemServiceImpl;
}
```

+ 创建真实的对象
> ExternalSystemServiceImpl
+ 默认调用真实函数
+ mock函数	
> mock ExternalSystemServiceImpl的retrieveGamingInfoByCms函数:
```
//非void函数
Mockito.doReturn(Object toBeReturned).when(T mock).retrieveGamingInfoByCms(Mockito.any());

//void 函数
Mockito.doCallRealMethod().when(externalSystemServiceImpl).retrieveCreditInfo();
```

#Logic Function

+ ######mock data
>> 边界数据: 0、-1等边界条件的数据,引起异常的数据(null)

+ ######mock function
>> mock 所有使用到的外部函数:
	Input/Output/Exception

+ ######assert result
	判断函数是否达到预期效果:
>>	org.junit.Assert
>>  org.junit.internal.ComparisonCriteria

```
   @Test
    public void assertQueryCustProfileNormals() {
        assertQueryCustProfileNormal("cxmid", custProfileBean, custInviteCardBean, custGrCardBean, Collections.singletonList(custContactBean));
    }

    private void assertQueryCustProfileNormal(String customerId, CustProfileBean assertProfileBean, CustInviteCardBean
            assertInviteCardBean, CustGrCardBean
                                      assertCustGrCardBean,
                              List<CustContactBean> assertContactBeanList) {

        ComparisonCriteria comparisonCriteria = new BeanPropertyComparisonCriteria();

        QueryCustProfileRequest request = new QueryCustProfileRequest();
        request.setCustomerId(customerId);
        QueryCustProfileResponse response = custProfileManageService.queryCustProfileInfo(request);
		
		//Assert
        Assert.assertEquals(customerId, response.getCustomerId());

        //ComparisonCriteria
        comparisonCriteria.arrayEquals("getProfileBean", new Object[]{assertProfileBean}, new Object[]{response.getProfileBean()});
        comparisonCriteria.arrayEquals("getInviteCardBean", new Object[]{assertInviteCardBean}, new Object[]{response.getInviteCardBean()});
        comparisonCriteria.arrayEquals("getCustGrCardBean", new Object[]{assertCustGrCardBean}, new Object[]{response.getCustGrCardBean()});

        comparisonCriteria.arrayEquals("getContactBeanList", assertContactBeanList.toArray(), response.getContactBeanList().toArray());
    }
```

#RESTful Interface

+ ######Mock MVC:
```
private MockMvc mockMvc;

@Before
public void beforeTest() {
    mockMvc = MockMvcBuilders.standaloneSetup(userFavoriteCustController).build();
}
```

+ ######Input

	org.springframework.test.web.servlet.MockMvc

>>> ######GET：
```
mockMvc.perform(MockMvcRequestBuilders.get("/rwscxm/api/v1/user/favorite/favorite_customers?userId={userId}&fuzzyName={fuzzyName}&sortType={sortType}", "1", "cmx", "1"))
```

>>> ######POST:
```
mockMvc.perform(MockMvcRequestBuilders.post("/rwscxm/api/v1/user/favorite/add_favorite_customers")
.content("{userId: \"1\",customerId: \"1\",customerIdList: [\"1\",\"2\"]}"))
```

>>> ######HEAD:
```
mockMvc.perform(MockMvcRequestBuilders.post("/rwscxm/api/v1/user/favorite/add_favorite_customers")
.content("{userId: \"1\",customerId: \"1\",customerIdList: [\"1\",\"2\"]}").
.contentType(MediaType.ALL)
.header("head1", "head2"))
```


+Output/Assert:

	org.springframework.test.web.servlet.MockMvc.andExpect()

>> ######status:
```
andExpect(MockMvcResultMatchers.status().isOk())
```

>> ######body(json):
```
andExpect(MockMvcResultMatchers.jsonPath(String expression, Matcher<T> matcher))

andExpect(MockMvcResultMatchers.jsonPath("$.result.customerInfoList[0].customerInfo[0].customerId", Matchers.is("1")));
```
>>>>>> ######expression:

>>>>>>$: root

>>>>>>$.a: 属性a

>>>>>>$.a[0]: 列表


#Database Function

+ 唯一的依赖
>> 外部数据库(无法mock)
+ 不能污染数据库
>> 事务回滚
>>>> pring-test: \@TransactionConfiguration
