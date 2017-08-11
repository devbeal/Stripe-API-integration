<?php
session_start();

$stripe = new StripeOfferAPI();
$stripe->createCustomer();

class StripeOfferAPI
{	
	
	const testApiKey = 'rgin9348guisdmf88fdm';
	
	//Create Customer
	public static function createCustomer()
	{
		
		$customerUrl = 'https://api.stripe.com/v1/customers';
		$headers = array('Authorization: Bearer '.self::testApiKey);
		
		$params = (object) array();
		$params->email = $_POST['emailAddress'];
		
		$params->shipping = (object) array();
		$params->shipping->name = $_POST['firstName'].' '.$_POST['lastName'];
		$params->shipping->phone = $_POST['phoneNumber'];
		
		$params->shipping->address = (object) array();
		$params->shipping->address->line1 = !empty($_POST['shipAddress']) ? $_POST['shipAddress'] : $_POST['address'];
		$params->shipping->address->city = !empty($_POST['shipCity']) ? $_POST['shipCity'] : $_POST['city'];
		$params->shipping->address->postal_code = !empty($_POST['shipPostalCode']) ? $_POST['shipPostalCode'] : $_POST['postalCode'];
		$params->shipping->address->state = !empty($_POST['shipState']) ? $_POST['shipState'] : $_POST['state']; 
		$params->shipping->address->country = 'US';
		
		$ch = curl_init($customerUrl);
		curl_setopt($ch,CURLOPT_HTTPHEADER,$headers);
		curl_setopt($ch,CURLOPT_POSTFIELDS,http_build_query($params));
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
		$response = curl_exec($ch);
		curl_close($ch);
				
		$response = json_decode($response,true);
		if($response['id'] && $response['created'])
		{
			$responseParams = (object) array();
			$responseParams->customerId = $response['id'];
			$_SESSION['customerId'] = $response['id'];
			$_SESSION['emailAddress'] = $_POST['emailAddress'];
			StripeOfferAPI::createOrder($responseParams);
		}
		else
			StripeOfferAPI::errorMsg($response);
	}
	
	//Create Offer Order
	public static function createOrder($responseParams)
	{
		
		$orderUrl = 'https://api.stripe.com/v1/orders';
		$headers = array('Authorization: Bearer '.self::testApiKey);
	
		$offer = (object) 
		array('amount' => StripeOfferAPI::product($_POST['productId'])->amount * 100,
			  'currency' => 'usd',
			  'description' => StripeOfferAPI::product($_POST['productId'])->name,
			  'parent' => StripeOfferAPI::product($_POST['productId'])->sku,
			  'quantity' => '1',
			  'type' => 'sku');
		
		$params = (object) array();
		$params->currency = 'usd';
		$params->customer = $responseParams->customerId;
		
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
			$responseParams->orderId = $response['id'];
			$_SESSION['offerOrderId'] = $response['id'];
			$_SESSION['offerProductName'] = StripeOfferAPI::product($_POST['productId'])->name;
			$_SESSION['offerAmount'] = StripeOfferAPI::product($_POST['productId'])->amount * 100;
			StripeOfferAPI::createToken($responseParams);
		}
		else
			StripeOfferAPI::errorMsg($response);
	}
	
	//Create Card Token
	public static function createToken($responseParams)
	{
				
		$tokenUrl = 'https://api.stripe.com/v1/tokens';
		$headers = array('Authorization: Bearer '.self::testApiKey);
		
		$params = (object) array();
		$params->card = (object) array();
		$params->card->number = $_POST['cardNumber'];
		$params->card->exp_month = $_POST['cardMonth'];
		$params->card->exp_year = $_POST['cardYear'];
		$params->card->cvc = $_POST['cardSecurityCode'];
		$params->card->name = $_POST['firstName'].' '.$_POST['lastName'];
		$params->card->address_line1 = $_POST['address'];
		$params->card->address_city = $_POST['city'];
		$params->card->address_zip = $_POST['postalCode'];
		$params->card->address_state = $_POST['state'];
		$params->card->address_country = 'US';
			
		$ch = curl_init($tokenUrl);
		curl_setopt($ch,CURLOPT_HTTPHEADER,$headers);
		curl_setopt($ch,CURLOPT_POSTFIELDS,http_build_query($params));
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
		$response = curl_exec($ch);
		curl_close($ch);
		
		$response = json_decode($response,true);
		if($response['id'] && $response['created'])
		{
			$responseParams->cardToken = $response['id'];
			StripeOfferAPI::updateCustomer($responseParams);
		}
		else
			StripeOfferAPI::errorMsg($response);
	}
	
	//Update Customer with Source
	public static function updateCustomer($responseParams)
	{
		
		$updateUrl = 'https://api.stripe.com/v1/customers/'.$responseParams->customerId;
		$headers = array('Authorization: Bearer '.self::testApiKey);
		
		$params = (object) array();
		$params->source = $responseParams->cardToken;
		
		$ch = curl_init($updateUrl);
		curl_setopt($ch,CURLOPT_HTTPHEADER,$headers);
		curl_setopt($ch,CURLOPT_POSTFIELDS,http_build_query($params));
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
		$response = curl_exec($ch);
		curl_close($ch);
		
		$response = json_decode($response,true);
		if($response['id'] && $response['created'])
		{
			$responseParams->cardToken = $response['id'];
			StripeOfferAPI::createCharge($responseParams);
		}
		else
			StripeOfferAPI::errorMsg($response);
	}
	
	//Create Offer Charge
	public static function createCharge($responseParams)
	{
		
		$chargeUrl = 'https://api.stripe.com/v1/charges';
		$headers = array('Authorization: Bearer '.self::testApiKey);
		
		$params = (object) array();
		$params->amount = StripeOfferAPI::product($_POST['productId'])->amount * 100;
		$params->currency = 'usd';
		$params->capture = 'true';
		$params->customer = $responseParams->customerId;
		$params->order = $responseParams->orderId;
		
		$ch = curl_init($chargeUrl);
		curl_setopt($ch,CURLOPT_HTTPHEADER,$headers);
		curl_setopt($ch,CURLOPT_POSTFIELDS,http_build_query($params));
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
		$response = curl_exec($ch);
		curl_close($ch);
		
		$response = json_decode($response,true);
		if($response['id'] && $response['created'])
		{
			header('Location: https://sigmaflashlight.com/sb/u2/index.php');
			exit();
		}
		else
			StripeOfferAPI::errorMsg($response);
	}
		
	//get product details
	public static function product($productId)
	{

		$product = (object) array();
		switch($productId)
		{
			case 'SFL1':
				$product->name = 'Sigma Flashlight 1x';
				$product->sku = 'SFL1';
				$product->amount = '56';
				break;
			case 'SFL2':
				$product->name = 'Sigma Flashlight 2x';
				$product->sku = 'SFL2';
				$product->amount = '97';
				break;
			case 'SFL3':
				$product->name = 'Sigma Flashlight 3x';
				$product->sku = 'SFL3';
				$product->amount = '117';
				break;
			case 'SFL5':
				$product->name = 'Sigma Flashlight 5x';
				$product->sku = 'SFL5';
				$product->amount = '145';
				break;
			case'SFL10':
				$product->name = 'Sigma Flashlight 10x';
				$product->sku = 'SFL10';
				$product->amount = '350';
				break;
			case'SFL15':
				$product->name = 'Sigma Flashlight 15x';
				$product->sku = 'SFL15';
				$product->amount = '525';
				break;
			case'SFL20':
				$product->name = 'Sigma Flashlight 20x';
				$product->sku = 'SFL20';
				$product->amount = '700';
				break;
			default:
				die("Please select a Product");
		}
	
		return $product;
	}
	
	//error handling
	public static function errorMsg($response)
	{
		
		$errMsg = $response['error']['message'];
		if(!empty($errMsg))
		{
			$queryParams = array('errMsg'=>$errMsg);
			$redirectUrl = "https://sigmaflashlight.com/sb/c/index.php?".http_build_query($queryParams);	
			if(!headers_sent())
				header("Location: ".$redirectUrl);
			
			die("Please contact technical support");
		}
	}
}
