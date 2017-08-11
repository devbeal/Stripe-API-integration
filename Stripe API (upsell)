<?php
session_start();

$stripe = new StripeUpsellAPI();
if($_SESSION['customerId'])
	$stripe->createOrder();

class StripeUpsellAPI
{	
	const testApiKey = "grekg893jg389frekf";

	//Create Upsell Order
	public static function createOrder()
	{
		
		$orderUrl = 'https://api.stripe.com/v1/orders';
		$headers = array('Authorization: Bearer '.self::testApiKey);

		$offer = (object) 
		array('amount' => StripeUpsellAPI::upsellProduct($_POST['upsellProductId'])->amount * 100,
			  'currency' => 'usd',
			  'description' => StripeUpsellAPI::upsellProduct($_POST['upsellProductId'])->name,
			  'parent' => StripeUpsellAPI::upsellProduct($_POST['upsellProductId'])->sku,
			  'quantity' => '1',
			  'type' => 'sku');
		
		$params = (object) array();
		$params->currency = 'usd';
		$params->customer = $_SESSION['customerId'];
		
		$params->items =  array();
		$params->items[] = $offer;
		
		$ch = curl_init($orderUrl);
		curl_setopt($ch,CURLOPT_HTTPHEADER,$headers);
		curl_setopt($ch,CURLOPT_POSTFIELDS,http_build_query($params));
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
		$response = curl_exec($ch);
		curl_close($ch);
		
		$response = json_decode($response,true);
		if($response['id'] && $response['created'])
		{
			$responseParams = (object) array();
			$responseParams->upsellOrderId = $response['id'];
			$_SESSION['upsellOrderId'] = $response['id'];
			$_SESSION['upsellProductName'] = StripeUpsellAPI::upsellProduct($_POST['upsellProductId'])->name;
			$_SESSION['upsellAmount'] = StripeUpsellAPI::upsellProduct($_POST['upsellProductId'])->amount * 100;

			StripeUpsellAPI::createCharge($responseParams);
		}
		else
			StripeUpsellAPI::errorMsg($response);
	}
		
	//Create Upsell Charge
	public static function createCharge($responseParams)
	{
		
		$chargeUrl = 'https://api.stripe.com/v1/charges';
		$headers = array('Authorization: Bearer '.self::testApiKey);
		
		$params = (object) array();
		$params->amount = StripeUpsellAPI::upsellProduct($_POST['upsellProductId'])->amount * 100;
		$params->currency = 'usd';
		$params->capture = 'true';
		$params->customer = $_SESSION['customerId'];
		$params->order = $responseParams->upsellOrderId;
		
		$ch = curl_init($chargeUrl);
		curl_setopt($ch,CURLOPT_HTTPHEADER,$headers);
		curl_setopt($ch,CURLOPT_POSTFIELDS,http_build_query($params));
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
		$response = curl_exec($ch);
		curl_close($ch);
		
		$response = json_decode($response,true);
		if($response['id'] && $response['created'])
		{
			header('Location: https://sigmaflashlight.com/sb/t/index.php');
			exit();
		}
 		else
			StripeOfferAPI::errorMsg($response);
	}
		
	//get upsell product details
	public static function upsellProduct($upsellProductId)
	{

		$upsellProduct = (object) array();
		switch($upsellProductId)
		{
			case '1year':
				$upsellProduct->name = '1 Year Warranty';
				$upsellProduct->sku = '1year';
				$upsellProduct->amount = '15';
				break;
			case '3year':
				$upsellProduct->name = '3 Year Warranty';
				$upsellProduct->sku = '3year';
				$upsellProduct->amount = '20';
				break;
			default:
				die("Please select a Product");
		}
		return $upsellProduct;
	}
	
	//error handling
	public static function errorMsg($response)
	{
	
		$errMsg = $response['error']['message'];
		if(!empty($errMsg))
		{
			$queryParams = array('errMsg'=>$errMsg);
			$redirectUrl = "https://sigmaflashlight.com/sb/u2/index.php?".http_build_query($queryParams);	
			if(!headers_sent())
				header("Location: ".$redirectUrl);
			
			die("Please contact technical support");
		}
	}
}
