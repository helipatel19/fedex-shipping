## Fedex ship service Integration in Laravel 7

This tutorial will guide you to integrate fedex shipping gateway in laravel 7 application.Here, we will use PHP FedEx API Wrapper library to integrate fedex.This library provides a fluid interface for constructing requests to the FedEx web service API.

**Step:1**

Install fresh laravel project by following command:

    composer create-project --prefer-dist laravel/laravel fedexIntegeration.

**Step:2**

Create fedex developer account to generate REST API credentials.we will need four credentials that we'll need to retrieve :

    1. FedEx account number
    2. fedEx meter number
    3. FedEx access key
    4. FedEx access key password
 
 add these credentials into `.env` file as below.

    FEDEX_KEY=developer_test_key
    FEDEX_API_PASSWORD=test_password
    FEDEX_ACCOUNT_NUMBER=fedex_test_account_number
    FEDEX_METER_NUMBER=fedex_test_meter_number
    
Next, create `fedex.php` file inside the config directory and add following code:

    return [
        'key'      => env('FEDEX_KEY'),
        'password' => env('FEDEX_API_PASSWORD'),
        'account'  => env('FEDEX_ACCOUNT_NUMBER'),
        'meter'    => env('FEDEX_METER_NUMBER'),
        'beta'     => env('FEDEX_USE_BETA', false),
    ];

Install JeremyDunn/php-fedex-api-wrapper by following command:

    composer require jeremy-dunn/php-fedex-api-wrapper
    
 **Step:3**
 
we have created two services of FedEx web service API.
See the official FedEx web service API documentation for a description of these services.

1. Fedex ship service.
2. Fedex Open shipment service.

Let's understand integration of Fedex Open shipment service. Firstly, create FedexOpenShipmentController by following command:

    php artisan make:controller FedexOpenShipmentController
 
 Next, configure fedex credentials inside the construct method of FedexOpenShipmentController.
    
        public function __construct() {
              $this->middleware('auth');
              $this->key = config('fedex.key');
              $this->password = \config('fedex.password');
              $this->meter_number = config('fedex.meter');
              $this->account_no = config('fedex.account');
        }

Then, create `openShipmentServiceProcess()` method and add following code:
    
    $shipDate = new \DateTime();
    
           $createOpenShipmentRequest = new ComplexType\CreateOpenShipmentRequest();
    // web authentication detail
           $createOpenShipmentRequest->WebAuthenticationDetail->UserCredential->Key = $this->key;
           $createOpenShipmentRequest->WebAuthenticationDetail->UserCredential->Password = $this->password;
    // client detail
           $createOpenShipmentRequest->ClientDetail->MeterNumber = $this->meter_number;
           $createOpenShipmentRequest->ClientDetail->AccountNumber = $this->account_no ;
    // version
           $createOpenShipmentRequest->Version->ServiceId = 'ship';
           $createOpenShipmentRequest->Version->Major = 15;
           $createOpenShipmentRequest->Version->Intermediate = 0;
           $createOpenShipmentRequest->Version->Minor = 0;
    
    // package 1
           $requestedPackageLineItem1 = new ComplexType\RequestedPackageLineItem();
           $requestedPackageLineItem1->SequenceNumber = 1;
           $requestedPackageLineItem1->ItemDescription = 'Product description 1';
           $requestedPackageLineItem1->Dimensions->Width = 10;
           $requestedPackageLineItem1->Dimensions->Height = 10;
           $requestedPackageLineItem1->Dimensions->Length = 15;
           $requestedPackageLineItem1->Dimensions->Units = SimpleType\LinearUnits::_IN;
           $requestedPackageLineItem1->Weight->Value = 2;
           $requestedPackageLineItem1->Weight->Units = SimpleType\WeightUnits::_LB;
    
    // requested shipment
           $createOpenShipmentRequest->RequestedShipment->DropoffType = SimpleType\DropoffType::_REGULAR_PICKUP;
           $createOpenShipmentRequest->RequestedShipment->ShipTimestamp = $shipDate->format('c');
           $createOpenShipmentRequest->RequestedShipment->ServiceType = SimpleType\ServiceType::_FEDEX_2_DAY;
           $createOpenShipmentRequest->RequestedShipment->PackagingType = SimpleType\PackagingType::_YOUR_PACKAGING;
           $createOpenShipmentRequest->RequestedShipment->LabelSpecification->ImageType = SimpleType\ShippingDocumentImageType::_PDF;
           $createOpenShipmentRequest->RequestedShipment->LabelSpecification->LabelFormatType = SimpleType\LabelFormatType::_COMMON2D;
           $createOpenShipmentRequest->RequestedShipment->LabelSpecification->LabelStockType = SimpleType\LabelStockType::_PAPER_4X6;
           $createOpenShipmentRequest->RequestedShipment->RateRequestTypes = [SimpleType\RateRequestType::_PREFERRED];
           $createOpenShipmentRequest->RequestedShipment->PackageCount = 1;
           $createOpenShipmentRequest->RequestedShipment->RequestedPackageLineItems = [$requestedPackageLineItem1];
    
    // requested shipment shipper
           $createOpenShipmentRequest->RequestedShipment->Shipper->AccountNumber = $this->account_no;
           $createOpenShipmentRequest->RequestedShipment->Shipper->Address->StreetLines = ['1202 Chalet Ln'];
           $createOpenShipmentRequest->RequestedShipment->Shipper->Address->City = 'Harrison';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Address->StateOrProvinceCode = 'AR';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Address->PostalCode = '72601';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Address->CountryCode = 'US';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Contact->CompanyName = 'Company Name';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Contact->PersonName = 'Person Name';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Contact->EMailAddress = 'shipper@example.com';
           $createOpenShipmentRequest->RequestedShipment->Shipper->Contact->PhoneNumber = '1-123-123-1234';
    
    // requested shipment recipient
            $createOpenShipmentRequest->RequestedShipment->Recipient->AccountNumber = '510051408';
            $createOpenShipmentRequest->RequestedShipment->Recipient->Address->StreetLines = ['2000 Freight LTL Testing'];
           $createOpenShipmentRequest->RequestedShipment->Recipient->Address->City = 'Harrison';
           $createOpenShipmentRequest->RequestedShipment->Recipient->Address->StateOrProvinceCode = 'AR';
           $createOpenShipmentRequest->RequestedShipment->Recipient->Address->PostalCode = '72601';
           $createOpenShipmentRequest->RequestedShipment->Recipient->Address->CountryCode = 'US';
           $createOpenShipmentRequest->RequestedShipment->Recipient->Contact->PersonName = 'John Doe';
           $createOpenShipmentRequest->RequestedShipment->Recipient->Contact->EMailAddress = 'recipient@gmail.com';
           $createOpenShipmentRequest->RequestedShipment->Recipient->Contact->PhoneNumber = '1-321-321-4321';
    
    // shipping charges payment
           $createOpenShipmentRequest->RequestedShipment->ShippingChargesPayment->Payor->ResponsibleParty = $createOpenShipmentRequest->RequestedShipment->Shipper;
           $createOpenShipmentRequest->RequestedShipment->ShippingChargesPayment->PaymentType = SimpleType\PaymentType::_SENDER;
    
    // send the create open shipment request
           $openShipServiceRequest = new Request();
           $createOpenShipmentReply = $openShipServiceRequest->getCreateOpenShipmentReply($createOpenShipmentRequest);
    
    // shipment is created and we have an index number
           $index = $createOpenShipmentReply->Index;
    
    
           /********************************
            * Add a package to open shipment
            ********************************/
           $addPackagesToOpenShipmentRequest = new ComplexType\AddPackagesToOpenShipmentRequest();
    
    // set index
           $addPackagesToOpenShipmentRequest->Index = $index;
    
    // reuse web authentication detail from previous request
           $addPackagesToOpenShipmentRequest->WebAuthenticationDetail = $createOpenShipmentRequest->WebAuthenticationDetail;
    
    // reuse client detail from previous request
           $addPackagesToOpenShipmentRequest->ClientDetail = $createOpenShipmentRequest->ClientDetail;
    
    // reuse version from previous request
           $addPackagesToOpenShipmentRequest->Version = $createOpenShipmentRequest->Version;
    
    // requested package line item
           $requestedPackageLineItem2 = new ComplexType\RequestedPackageLineItem();
           $requestedPackageLineItem2->SequenceNumber = 2;
           $requestedPackageLineItem2->ItemDescription = 'New package added to open shipment';
           $requestedPackageLineItem2->Dimensions->Width = 20;
           $requestedPackageLineItem2->Dimensions->Height = 10;
           $requestedPackageLineItem2->Dimensions->Length = 12;
           $requestedPackageLineItem2->Dimensions->Units = SimpleType\LinearUnits::_IN;
           $requestedPackageLineItem2->Weight->Value = 4;
           $requestedPackageLineItem2->Weight->Units = SimpleType\WeightUnits::_LB;
           $addPackagesToOpenShipmentRequest->RequestedPackageLineItems = [$requestedPackageLineItem2];
    
    // send the add packages to open shipment request
    //       $addPackagesToOpenShipmentReply = $openShipServiceRequest->getAddPackagesToOpenShipmentReply($addPackagesToOpenShipmentRequest);
    
           /************************************
            * Retrieve the open shipment details
            ************************************/
           $retrieveOpenShipmentRequest = new ComplexType\RetrieveOpenShipmentRequest();
           $retrieveOpenShipmentRequest->Index = $index;
           $retrieveOpenShipmentRequest->WebAuthenticationDetail = $createOpenShipmentRequest->WebAuthenticationDetail;
           $retrieveOpenShipmentRequest->ClientDetail = $createOpenShipmentRequest->ClientDetail;
           $retrieveOpenShipmentRequest->Version = $createOpenShipmentRequest->Version;
    
    //       $retrieveOpenShipmentReply = $openShipServiceRequest->getRetrieveOpenShipmentReply($retrieveOpenShipmentRequest);
    
           /***********************
            * Confirm open shipment
            ***********************/
           $confirmOpenShipmentRequest = new ComplexType\ConfirmOpenShipmentRequest();
           $confirmOpenShipmentRequest->WebAuthenticationDetail = $createOpenShipmentRequest->WebAuthenticationDetail;
           $confirmOpenShipmentRequest->ClientDetail = $createOpenShipmentRequest->ClientDetail;
           $confirmOpenShipmentRequest->Version = $createOpenShipmentRequest->Version;
           $confirmOpenShipmentRequest->Index = $index;
    
           $confirmOpenShipmentReply = $openShipServiceRequest->getConfirmOpenShipmentReply($confirmOpenShipmentRequest);

    // store the label into storage folder
    
            Storage::put('public/storage/label.pdf', $confirmOpenShipmentReply->CompletedShipmentDetail->CompletedPackageDetails[0]->Label->Parts[0]->Image);
    
    // retrieve the content from label.pdf
    
            return response()->file(
                public_path('storage/storage/label.pdf')
            );
    
Now, create a ` openShipService.blade.php` view for fedex open shipment service and add the following code into it.

    @extends('layouts.app')
    @section('content')
        <div class="container">
            <div class="modal-title">
                <h2>Fedex Open ship service</h2>
            </div>
            <div class="nav-link">
                <a href="{{ route('openShipService') }}" class="btn btn-lg btn-primary">Demo  Openship service</a>
            </div>
        </div>
    @endsection

Next, create index() method into FedexOpenShipmentController to return the view.

    public function index(){
          return view('openShipService');
    }
        
Then, define routes into `routes/web.php` file.

    Route::get('/view-openship-service', 'FedexOpenShipmentController@index')->name('view-openship-service');
    
    Route::get('/openShipService', 'FedexOpenShipmentController@openShipmentServiceProcess')->name('openShipService');


Finally, you will see the label containing details about open shipment service reply as below:
    
   ![Openshipment service Label](public/storage/storage/fedex_open_ship_service.png)


