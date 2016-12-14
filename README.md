<?php
namespace mobile\controllers;

use Yii;
use yii\web\Controller;
use yii\filters\VerbFilter;
use yii\filters\AccessControl;
use vendor_libs\proxy\ShuidaoProxy;

/**
 * Base controller
 */
class BaseController extends Controller
{
    public $_user = array();
    public $_token = '';

    /**
     * @inheritdoc
     */
    public function behaviors()
    {/*{{{*/
        return [
        ];
    }/*}}}*/

    /**
     * @inheritdoc
     */
    public function actions()
    {/*{{{*/
        return [
        ];
    }/*}}}*/

    /**
     * @inheritdoc
     */
    public function beforeAction($action)
    {/*{{{*/
        if (parent::beforeAction($action)) {
            $this->_token = isset($_COOKIE['dt_token']) ? urlencode($_COOKIE['dt_token']) : '';
            $user_info = [];

            try{
                $user_info = $this->_token
                    ? ShuidaoProxy::run('v1', 'user', 'info', [], [], 'dt_token=' . $this->_token)
                    : [];
            } catch (\Exception $e) {}

            if(empty($user_info) || (isset($user_info['code']) && $user_info['code'])){
                $this->_user['_id'] = '';
                $this->_user['name'] = '';
                $this->_user['company_id'] = '';
                $this->_user['company_name'] = '';
                $this->_user['has_purchase'] = '';
                $this->_user['department'] = '';
                $this->_user['role'] = '';
            }else{
                $this->_user = $user_info['res']['user'];
            }

            return true;
        }

        return false;
    }/*}}}*/

    public function renderPartial($view, $params = [])
    {/*{{{*/
        $default_params = ['user' => $this->_user, 'is_login' => isset($this->_user['_id'])];
        $params = array_merge($default_params, $params);
        return $this->getView()->render($view, $params, $this);
    }/*}}}*/

    public function render($view, $params = [])
    {/*{{{*/
        $default_params = ['user' => $this->_user, 'is_login' => isset($this->_user['_id']), 'api_host' => Yii::$app->params['apiHost'], 'company_name' => $this->_user['company_name']];
        $params = array_merge($default_params, $params);
        return $this->getView()->render($view, $params, $this);
    }/*}}}*/

}
