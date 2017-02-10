# 特卖资源访问层

## 部署
### git clone 项目
          cd Global/General/
          git clone git@code.byted.org:temai/Ral.git
  部署在 Global/General/Ral/
### 在所在的yaf项目的BootStrap.php中加入如下代码
      public function _initDoctrine(Dispatcher $dispatcher){
        $environ = ini_get('yaf.environ');
        if(in_array($environ,array('menghuanbiao','hanyakun','kongdeshi','chenlei','liuzhenkun','huanggaomin','hefeng','qiluyan'))){
          $composer_autoload = "/data00/test_users/{$environ}/web/Global/General/Ral/vendor/autoload.php";
        }   
        else{
          $composer_autoload = '/opt/install/Global/General/Ral/vendor/autoload.php';
        }   
        if(is_file($composer_autoload)){
          require_once $composer_autoload;
        }   
      }   
### 若不在yaf项目中使用请 require_once Ral/vendor/autoload.php 先

## 获取id生成器
        use General\Ral\CommonFactory;
        $id_alloc = CommonFactory::getInstance(CommonFactory::SOURCE_TYPE_IDALLOC);
        $ids = $id_alloc->get(10);
        $id = $id_alloc->get();
  
## 获取&操作redis
         use General\Ral\CommonFactory;
         $redis = CommonFactory::getInstance(CommonFactory::SOURCE_TYPE_REDIS);
         $ress->hmset('hkey',array('a'=>'1','b'=>'2'));
  
## 获取&操作数据库
        use General\Ral\CommonFactory;
        use General\Ral\Match;

        $database = CommonFactory::getInstance(CommonFactory::SOURCE_TYPE_MYSQL);
        $match =Match::getInstance();
        $matches = Match::andX(
            Match::gt('id',100001300)
        );
        $match->where($matches);
        $match->limit(10);
        $match->offset(10);
        $match->orderBy(array('id' => Match::ASC));
        $conn = array(
            'driver'   => 'pdo_mysql',
            'user'     => 'root',
            'password' => 'root',
            'dbname'   => 'smart',
            'host'     => '10.4.40.23',
            'port'     => 3313,
        );
        $res = $database->table('address',$conn)->matching($match)->get(array('id','userId','province','city','area','mobile','code'));
        Match 中支持多种方法，如in，lt, gt,where,andWhere,orWhere 等
        eg: where (id>100001300 and id <100005300) or title like '%喜欢% order by id asc limit 10,10'
        $match = Match::getInstance();
        $matches = Match::andX(
                Match::gt('id',100001300),
                    Match::le('id',100005300)
                );
        $match->where($matches);
        $match_like = Match::like('title','喜欢');
        $match->andWhere($match_like);
        $match->limit(10);
        $match->offset(10);
        $match->orderBy(array('id' => Match::ASC));
        
### 更新
 
        sql: update t_assess set title='测试测试' where ...
        $res = $database->table('assess')->matching($match)->set(
                   array('title'=>'测试测试')
        );

### 增加
 
        sql: insert into t_assess values('测试测试',....);
        $res = $database->table('assess')->set(
                   array('title'=>'测试测试')
        );

### 批量添加
 
        sql: insert into t_assess values('测试测试',....),('测试测试',....)
        $res = $database->table('assess')->mset(
                       array(array('title'=>'测试测试'),
                       array('title'=>'测试测试'))
        );

### 删除
 
        sql: delete from  t_assesswhere ...
        $res = $database->table('assess')->matching($match)->delete();
### 事务的例子
 
        $conn = array(
            'driver'   => 'pdo_mysql',
            'user'     => 'root',
            'password' => 'root',
            'dbname'   => 'temaidb',
            'host'     => '10.2.0.50',
            'port'     => 3307,
        );

        try{
            $database->master()->table('assess',$conn)->beginTransactions();
            $res = $database->table('assess',$conn)->matching($match)->set(array('title'=>'ral commit test rolllback 111'));
            $res = $database->table('assess',$conn)->matching($match)->get(array('title','id'));
            throw new \Exception('error occured');
            $database->commit();
        }
        catch (\Exception $e){
            $database->rollback();
        }
        $res = $database->table('assess',$conn)->matching($match)->get(array('title','id'));

## 单元测试
### 生成测试代码
    bash gen_entity_test.sh [product|user|order] entity_name
### 运行单元测试
    php Test.php
