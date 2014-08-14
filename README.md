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
