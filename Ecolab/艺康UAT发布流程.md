

#### :1st_place_medal: 切换到 develop 分支

```shell
git checkout develop

git pull

git status

```



#### :1st_place_medal: 切换到 uat-1.0.0 分支 

```shell
git checkout uat-1.0.0

git pull

git merge develop

git status

git push
```



#### :1st_place_medal: 到 gitlab 获取镜像





####  :1st_place_medal: 内网访问  10.246.194.69



账号：admin

密码:：CFSP@ssword#!



#### :1st_place_medal: 服务器密码  P@ssw0rd!!!!

```bash
sudo sh /data/eclb/repo/eclb-service/run.sh

tail -f /data/eclb/logs/eclb.log | grep '过滤数据'
```



### :1st_place_medal: 接口403

```html
http://10.246.125.108:8080/iam/v1/permission/fresh/hzero-platform?metaVersion=1.0.0

http://10.246.125.108:8080/iam/v1/permission/cache/hzero-platform


curl -X POST --header 'Content-Type: application/json' --header 'Accept: text/plain' --header 'Authorization: Bearer 93912bf9-1e2e-4356-b97d-7751313d2769' 'http://10.246.125.108.12:8080/iam/v1/permission/fresh/hzero-platform?metaVersion=1.0.0'


curl -X POST --header 'Content-Type: application/json' --header 'Accept: text/plain' --header 'Authorization: Bearer 93912bf9-1e2e-4356-b97d-7751313d2769' 'http://10.246.125.108:8080/iam/v1/permission/cache/hzero-platform'

```









```json


https://testcfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/approval/level/heads/copy?functionObject=CUST

https://testcfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/target/heads/copy?functionObject=CUST

https://testcfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/approval/level/heads/copy?functionObject=PRICE

https://testcfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/target/heads/copy?functionObject=PRICE


-- 

http://10.246.194.39:8080/ecsy/v1/0/system/workflow/approval/level/heads/copy?functionObject=CUST

http://10.246.194.39:8080/ecsy/v1/0/system/workflow/target/heads/copy?functionObject=CUST

http://10.246.125.108:8080/ecsy/v1/0/system/workflow/approval/level/heads/copy?functionObject=PRICE

http://10.246.125.108:8080/ecsy/v1/0/system/workflow/target/heads/copy?functionObject=PRICE



https://cfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/approval/level/heads/copy?functionObject=CUST
PRICE GROUP




https://cfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/approval/level/heads/copy?functionObject=PRICE

https://cfsapi.ecolab.com.cn/ecsy/v1/0/system/workflow/target/heads/copy?functionObject=PRICE
```



#### :1st_place_medal:BOM

```json
[
  {
    "materialId": 17,
    "materialSkuNum": "7503236",
    "materialNameZh": "50%硫酸溶液 200KG",
    "component": 18,
    "componentSkuNum": "7503912",
    "componentDescription": "50%乙二醇 1000KG",
    "quantity": 12,
    "tenantId": 0
  },
  {
    "materialId": 17,
    "materialSkuNum": "7503236",
    "materialNameZh": "50%硫酸溶液 200KG",
    "component": 19,
    "componentSkuNum": "7503232",
    "componentDescription": "78% 硫酸溶液 1000KG",
    "quantity": 13,
    "tenantId": 0
  }
]
```



```json
[
  {
    "materialId": 17,
    "materialSkuNum": "7503236",
    "materialNameZh": "50%硫酸溶液 200KG",
    "component": 18,
    "componentSkuNum": "7503912",
    "componentDescription": "50%乙二醇 1000KG",
    "quantity": 12,
    "tenantId": 0
  },
  {
    "id": 4,
    "materialId": 17,
    "materialSkuNum": "7503236",
    "materialNameZh": "50%硫酸溶液 200KG",
    "component": 98,
    "componentSkuNum": "7801313",
    "componentDescription": "F112 211KG",
    "quantity": 3,
    "tenantId": 0
  }
]
```



#### :1st_place_medal:SQL

```sql
SELECT
	market_segment
FROM
	eclb_contract
WHERE
	(
		unit_code = 'INST'
		AND (
			contract_status = 'WC'
			OR contract_id IN (
				SELECT
					c.contract_id
				FROM
					eclb_contract c
				WHERE
					c.contract_status = 'HY'
				AND c.creator IN (
					SELECT DISTINCT
						pos.`name`
					FROM
						eclb_service.ecmd_position pos
					WHERE
						pos.position_name = 'Contract Admin'
				)
			)
		)
		AND return_flag = 1
		AND tenant_id = 0
	)
ORDER BY
	contract_id ASC
```





#### :1st_place_medal: 循环依赖

```java
Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'IEcmaGroupApplyHeaderServiceImpl': Bean with name 'IEcmaGroupApplyHeaderServiceImpl' has been injected into other beans [ecmaGroupApplyHeaderServiceImpl] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.
```





```java
@PostMapping({"query"})
    public ResponseEntity<Page> listTask(
        @PathVariable @ApiParam(value = "租户ID",required = true) long organizationId, 
        @RequestBody @ApiParam(value = "待办事项查询条件",required = true) 
        CustomTaskQueryRequest request, 
        @RequestParam @ApiParam("暂时不知道这是干啥的") Map<String, String> requestParams) {
        
        String employeeCode = UserDetailsHelper.getEmployeeCode(organizationId);
        //明天尝试根据申请人查询流程id

        if(StringUtils.isEmpty(employeeCode)) {
            throw new CommonException("hwfl.security.error.user_not_relate_emp", new Object[0]);
        } else {
            request.setActive(Boolean.valueOf(true));
            request.setCandidateOrAssigned(employeeCode);
            request.setTenantId(String.valueOf(organizationId));

            if(request.getSort() == null) {
                request.setSort("createTime");
                request.setOrder("desc");
            }
            String assignName=request.getAssignee();
            Integer page=request.getPage().intValue();
            Integer size=request.getSize().intValue();
            if(request.getAssignee()!=null){
                request.setSize(Integer.MAX_VALUE);
                request.setAssignee(null);
            }
            /*if(request.getStartUserName()!=null){
                request.setSize(Integer.MAX_VALUE);
                assignName=request.getStartUserName();
            }*/
            //申请人过滤


            DataResponse dataResponse=this.activitiService.listTask(request, requestParams);
            List<TaskResponseExt> list = (List<TaskResponseExt>)dataResponse.getData();
            if(!CollectionUtils.isEmpty(list)&&!StringUtils.isEmpty(assignName)){
                list=list.stream().filter(a->a.getStartUserName().contains(assignName)).collect(Collectors.toList());
                dataResponse.setData(list.subList(page*size,list.size()>(page+1)*size?(page+1)*size:list.size()));
                dataResponse.setTotal(list.size());
            }


            //申请人过滤结束
            return Results.success(DataResponse2Page.translate(dataResponse,size));
        }
    }
```



:panda_face: :sparkles: :camel: :boom: :pig:





