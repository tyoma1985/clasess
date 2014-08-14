clasess
=======
<?php
abstract class Analytics
{
	public $returnedData;
	public $error;
	
	protected $session;
	
	protected function sendPost($url, $data)
	{
		$curl = curl_init($url);
		
		curl_setopt($curl, CURLOPT_POST, 1);
		curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
		curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
		
		$result = curl_exec($curl);
		curl_close($curl);
		
		return $result;
	}
	
	protected function createDataBlock($campaign, $clicks, $shows, $budget, $costType = 'cpc')
	{
		return array(
			'campaignID' => $campaign,
			'clicks' => $clicks,
			'shows' => $shows,
			'budget' => $budget,
			'costType' => $costType
		);
	}
	
	protected function writeError($code, $title, $desc = null)
	{
		$this->error = array(
			'source' => get_class($this),
			'code' => $code,
			'title' => $title,
			'desc' => $desc
		);
	}
}
<?php
final class Begun extends Analytics
{
	private $username = 'maxim@mango-media.ru';
	private $password = 'pS.mG7mA';
	private $token = '97b5972ccbdf567218f201b9df91df36';
	
	public function __construct()
	{
		$this->login();
	}
	
	private function login()
	{
		$data = array(
			'method' => 'Login',
			'params' => array(
				'login' => $this->username,
				'password' => $this->password,
			),
			'token' => $this->token
		);
		
		$send = $this->send($data);
		
		if(!$send) return false;
		
		$this->session = $this->returnedData->data->session_id;
	}
	
	public function getCampaignsList(array $idList = null)
	{
		if(empty($this->session)) 
		{
			$this->writeError(0, 'empty session id');
			
			return false;
		}
		
		$data = array(
			'method' => 'Ads.Campaign.GetList',
			'params' => array(
				//'status' => 'Active',
				'session_id' => $this->session,
				'token' => $this->token,
				'login' => $this->username,
			),
			'session_id' => $this->session,
			'token' => $this->token
		);
		/*
		$data = array(
			'method' => 'Ads.Campaign.GetStats',
			'params' => array(
				'campaign_id' => $idList,
				'report_type' => 'Day',
				'date_start' => date('Y-m-d', time() - 86400),
				'date_end' => date('Y-m-d')
			),
			'session_id' => $this->session,
			'token' => $this->token
		);
		*/
		$send = $this->send($data);
		
		if(!$send) return false;
		
		if(!empty($idList))
		{
			$newList = array();
			
			foreach($this->returnedData->data->list as $key => $campaign)
			{
				if(in_array($campaign->campaign_id, $idList)) $newList[] = $campaign;
			}
			
			$this->returnedData->data->list = $newList;
		}
		
		$newData = array();
		
		foreach($this->returnedData->data->list as $key => $campaign)
		{
			$newData = $this->createDataBlock($campaign->campaign_id, $campaign->clicks_yesterday, $campaign->shows_yesterday, $campaign->budget_yesterday, $campaign->cost_type);
		}
		
		return $newData;
	}
	
	private function send(array $data)
	{
		$json = json_encode($data);
		
		$returned = $this->sendPOST('https://smart.begun.ru/api/', $json);
		$returned = json_decode($returned);
		
		if(empty($returned->data)) 
		{
			if(isset($returned->error)) 
			{
				$code = $returned->error->error_code;
				$title = $returned->error->error_message;
				$desk = $returned->error->error_detail;
			}
			else 
			{
				$code = 0;
				$title = 'Unknown error';
				$desk = null;
			}
			
			$this->writeError($code, $title, $desc);
			
			return false;
		}
		else 
		{
			$this->returnedData = $returned;
			
			return true;
		}
	}
}
class Template
{
	private $template;
	public function load($templateUrl) 
	{ 
		if (is_readable($templateUrl))
		{
			return $this->template = file_get_contents($templateUrl);
		}
	}
	public function process($data)
	{
		foreach($data as $key => $value)
		{
			$this->template = str_replace("%$key%", htmlspecialchars ($value), $this->template);
		}
	}
	public function display()
	{
		echo $this->template;
	} 
}
