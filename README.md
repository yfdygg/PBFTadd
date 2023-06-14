# PBFTadd
pragma solidity ^0.4.25;
pragma experimental ABIEncoderV2;

import "./Table.sol";

contract LinkState {

    event CreateResult(int256 count);
    event InsertResult(int256 count);
    event UpdateResult(int256 count);
    event RemoveResult(int256 count);

    TableFactory tableFactory;
    string constant TABLE_NAME = "link_state";
    
    //权限控制部分
    address owner;//合约创建者    
    address[] super_user_list = new address[](10);//超级用户列表 address还是string类型
    address[] permissioned_user_list = new address[](10);//授权用户列表
    
    constructor() public {
    	//1个合约创建者
    	owner = msg.sender;
    	//初始10个超级用户，根据区块链节点数量确定，后续可能不变
    	super_user_list[0] = owner;
    	super_user_list[1] = 0x1840eea2abdb1d8981ec04def470cb726c9c2e87;//手动添加
    	super_user_list[2] = 0xf067f65c50bac9394d1d04961c1a6bef96781e0d;//手动添加
    	super_user_list[3] = 0x4818980730b591dd3da22f0de8659572aa26281f;//手动添加
    	super_user_list[4] = 0x827b750540c5905f3b1b736b8b19f0f34c58c625;//手动添加
    	super_user_list[5] = 0x4aca9fd10907bcc9aa89a5b6ea170b24c80e29dd;//手动添加
    	super_user_list[6] = 0xd4e4406087e7e956d3df648ca7ab4cdd23288e96;//手动添加
    	super_user_list[7] = 0x093d083891ceed1274ecd9e3f2263d69834487da;//手动添加
    	super_user_list[8] = 0x0b77a72023df0b13156dbdd96fdf51919d861fe6;//手动添加
    	super_user_list[9] = 0xb14542fdacd607b1a32fdb27538ea30ed7893ad0;//手动添加
    	
    	//初始10个授权用户，根据区块链节点数量确定初始数量，后续可删减
	permissioned_user_list[0] = super_user_list[0];
    	permissioned_user_list[1] = super_user_list[1];
    	permissioned_user_list[2] = super_user_list[2];
    	permissioned_user_list[3] = super_user_list[3];
    	permissioned_user_list[4] = super_user_list[4];
    	permissioned_user_list[5] = super_user_list[5];
    	permissioned_user_list[6] = super_user_list[6];
    	permissioned_user_list[7] = super_user_list[7];
    	permissioned_user_list[8] = super_user_list[8];
    	permissioned_user_list[9] = super_user_list[9];
    	
        tableFactory = TableFactory(0x1001); 
        // the parameters of createTable are tableName,keyField,"vlaueFiled1,vlaueFiled2,vlaueFiled3,..."
        tableFactory.createTable(TABLE_NAME, "source", "destination,delay,bandwidth,time");
    }
    
    //创建者修饰符
    modifier onlyOwner{
    	require(msg.sender == owner);
    	_;
    }
    //超级用户修饰符
    modifier superUser{
    	bool flag;
    	for(uint256 i = 0;i < super_user_list.length;++i){
    	    if(msg.sender == super_user_list[i]){
    	        flag = true;
    	    }
    	}
    	require(flag == true);
    	_;
    }
    //授权用户修饰符
    modifier permissionedUser{
        bool flag;
        for(uint256 i = 0;i < permissioned_user_list.length;++i){
    	    if(msg.sender == permissioned_user_list[i]){
    	        flag = true;
    	    }
    	}
    	require(flag == true);
    	_;
    }
    
    //超级用户>>>添加授权用户，成功返回0，失败返回-1
    function addPermissionedUser(address name) public superUser returns(int256){
    	if(permissioned_user_list.length < 10000){
    	    permissioned_user_list.push(name);
    	    return 0;
    	}
    	return -1;
    }
    //超级用户>>>删除授权用户，成功返回0，失败返回-1
    function deletePermissionedUser(address name) public superUser returns(int256){
    	if(permissioned_user_list.length > 0){
    	    for(uint256 i = 0;i < permissioned_user_list.length;++i){
    	    	if(permissioned_user_list[i] == name){
    	    	    delete permissioned_user_list[i]; //删除不干净，变为0x0000了，需要修改
    	    	    return 0;
    	    	}
    	    }
    	}	
    	return -1;
    }
    //授权用户>>>查阅授权用户列表
    function readPermissionedUser() public permissionedUser returns(address[] memory){
    	return permissioned_user_list;
    } 
    //合约创建者>>>调用自毁函数
    function destroy() public onlyOwner {
    	selfdestruct(owner);
    }
    
    //输出msgSender函数
    function msgSender() 
    public 
    view
    returns(address)
    {
    	return msg.sender;
    }
    
    //select records 输入source,destination，根据source,destination匹配，输出source,destination,delay,bandwidth,time
     function select(string memory source, string memory destination)
     public
     permissionedUser
     returns (string[5][] memory)
     {
         Table table = tableFactory.openTable(TABLE_NAME);
         
         Condition condition = table.newCondition();
         condition.EQ("source",source);
         condition.EQ("destination",destination);
	
	 //调用Table.sol中的select函数，根据key和condition筛选出entries
         Entries entries = table.select(source, condition);
        
         //source ip是string类型string[][]第一个参数列
         string[5][] memory results_bytes_list = new string[5][](uint256(entries.size()));
	//遍历entries中的entry
         for (int256 i = 0; i < entries.size(); ++i) {
             Entry entry = entries.get(i);
             	
             results_bytes_list[uint256(i)][0] = entry.getString("source");
             results_bytes_list[uint256(i)][1] = entry.getString("destination");
             results_bytes_list[uint256(i)][2] = entry.getString("delay");
             results_bytes_list[uint256(i)][3] = entry.getString("bandwidth");
             results_bytes_list[uint256(i)][4] = entry.getString("time");
         }
         return (results_bytes_list);
     }
    
    //insert records 输入source,destination,delay,bandwidth,time  返回数字成功与否
    function insert(string memory source,  string memory destination, int256 delay, int256 bandwidth, string memory time)
    public
    permissionedUser
    returns (int256)
    {
        Table table = tableFactory.openTable(TABLE_NAME);

        Entry entry = table.newEntry();
        entry.set("source", source);
        entry.set("destination", destination);
        entry.set("delay", delay);
        entry.set("bandwidth", bandwidth);
        entry.set("time", time);

        int256 count = table.insert(source, entry);
        emit InsertResult(count);

        return count;
    }
    
    //update records 输入source,destination,delay,bandwidth,time,根据source,destination,time匹配信息更新
    function update(string memory source,  string memory destination, int256 delay, int256 bandwidth, string memory time)
    public
    permissionedUser
    returns (int256)
    {
        Table table = tableFactory.openTable(TABLE_NAME);

        Entry entry = table.newEntry();
        entry.set("delay",delay );
        entry.set("bandwidth",bandwidth);

        Condition condition = table.newCondition();
        condition.EQ("source", source);
        condition.EQ("destination", destination);
        condition.EQ("time", time);

        int256 count = table.update(source, entry, condition);
        emit UpdateResult(count);

        return count;
    }
    //remove records 输入source,destination 根据source,destination匹配
    function remove(string memory source, string memory destination) 
    public 
    permissionedUser
    returns (int256) {
        Table table = tableFactory.openTable(TABLE_NAME);

        Condition condition = table.newCondition();
        condition.EQ("source",source);
        condition.EQ("destination",destination);

        int256 count = table.remove(source, condition);
        emit RemoveResult(count);

        return count;
    }
}
