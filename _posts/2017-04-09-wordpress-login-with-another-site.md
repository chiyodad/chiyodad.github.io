---
published: true
title: [Wordpress] 타사이트 사용자 정보를 통해 회원 생성 후 로그인하기
layout: post
---


### [Wordpress] 타사이트 사용자 정보를 통해 회원 생성 후 로그인하기 

A사이트의 회원 정보 그대로 B사이트에서 로그인 정보를 받아 회원으로 자동 등록 해주고 싶을 때가 있습니다. 이런 경우 보통 SSO로 처리하거나 DB를 복사하여 구성하는 경우가 많죠. 하지만 잠깐 쓰고 말 이벤트 사이트에서 그렇게 하기란 너무 번잡한 일입니다. 더욱이 A사이트가 Wordpress가 아니라면 DB구조도 달라 쉽지가 않죠.

아래 코드를 `function.php` 에 넣어주면 가능합니다.
하지만 그전에 A사이트 DB를 B사이트에서도 조회할 수 있도록 function을 만들어 주거나 DB를 열어주어야 합니다.

로그인 흐름은 다음과 같습니다.

1. 아이디 or 이메일을 조회하여 현재 B 사이트에 없다면
2. A 사이트에서 조회하여 
3. 회원 정보가 존재하면 B 사이트 회원으로 등록 후 
4. 로그인 상태로 첫페이지로 Redirect 해준다.

```php
// wp_authenticate action 을 통해 로그인시 hook 을 겁니다.
add_action( 'wp_authenticate' , 'my_check_authentication' );
function my_check_authentication ( $username ) {

        global $wpdb;

       //아이디나 이메일로 user 정보 확인
       $user_email = get_user_by('email', $username);
       $user_login = get_user_by('login', $username);

        //email, 아이디 둘다 없는 경우 
       if(!$user_email && !$user_login) {  
       
          $user_pwd = $_POST['pwd'];

          // A사이트 조회
          // 여기서는 A사이트 DB를 직접 접속하여 조회했습니다.
          // A사이트에서 조회할 수 있는 IP를 B사이트 서버 IP만 열어주면 좀 더 안전합니다.
          $sqluser   = "chiyodad";
          $sqlpwd    = "welcome";
          $sqldb     = "chiyodad_db";
          $sqlhost   = "127.0.0.1";

          $mysqli = new mysqli($sqlhost, $sqluser, $sqlpwd, $sqldb) or die("sql error.");
          $mysqli->autocommit(TRUE);
          $sql_query = "select * from user_info_v where (email ='".$username."' or user_id = '".$username."')  and user_pwd = password('".$user_pwd."') limit 1" ;
          // A사이트의 Password 를 MySQL password 함수로만 암호화 했을 때 유효합니다.

          $result = $mysqli->query($sql_query);
          $user_info = $result->fetch_assoc();
          $result->close();
          $mysqli->close();

          // A사이트에 사용자 정보가 존재한다면
          if($user_info) {   
             // B사이트 사용자로 등록하기
             $new_user = array(
                'user_login'  =>  $user_info['user_id'],
                'user_email'  =>  $user_info['email'],
                'user_pass'   =>  $user_pwd, // 입력받은 패스워드를 그대로 넣어줍니다.  
                'first_name'  =>  $user_info['user_name'], 
                'role'        =>  'subscriber' // 구독자로 초기 등급을 설정합니다.   
             );

             $user_id = wp_insert_user( $new_user ) ; // adding user to the database

            //등록 완료 시 로그인 및 홈페이지로 redirect
            if ( !is_wp_error( $user_id ) ) {                
                update_user_meta( $user_id, 'role', '기존회원'); // 특정 user_meta 값을 넣어주어야 할 때 
                wp_set_auth_cookie( $user_id); //신규 유저 로그인
                wp_redirect(site_url()); // 홈페이지로 redirect
            }
           }      
        return;
    }      
}
```

각 사이트 사정마다 내부 코드는 얼마든지 달라질 수 있습니다.
로그인 체크와 회원 등록에 초점을 맞추어 작성해봤습니다.
궁금한 점은 트윗(@chiyodad)으로 멘션주세요.
