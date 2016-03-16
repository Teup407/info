# Magento Code Snippets #

## Download extension manually using mage ##
```bash
./mage config-set preferred_state stable
./mage clear-cache
./mage sync
./mage download community Module_Name
```

## Reindex ##

```bash
php -f shell/indexer.php reindexall
```

```php
<?php
// clear cache
Mage::app()->removeCache('catalog_rules_dirty');
// reindex prices
Mage::getModel('index/process')->load(2)->reindexEverything();
/*
1 = Product Attributes
2 = Product Attributes
3 = Catalog URL Rewrites
4 = Product Flat Data
5 = Category Flat Data
6 = Category Products
7 = Catalog Search Index
8 = Tag Aggregation Data
9 = Stock Status
*/
```

## Delete cache/sessions ##
```bash
rm -rf var/log/*
rm -rf var/cache/*
rm -rf var/session/*
rm -rf var/report/*
rm -rf var/locks/*
rm -rf app/code/core/Zend/Cache
rm -rf var/ait_rewrite/*
rm -rf media/css/*
rm -rf media/js/*

// if there are many files which can't get deleted in once
find /path/to/session/* -mtime +1 -exec rm {} \;
```

## Load category by id ##

```php
<?php
$_category = Mage::getModel('catalog/category')->load(89);
$_category_url = $_category->getUrl();
```

## Load product by id or sku ##

```php
<?php
$_product_1 = Mage::getModel('catalog/product')->load(12);
$_product_2 = Mage::getModel('catalog/product')->loadByAttribute('sku','cordoba-classic-6-String-guitar');
```

## Get Configurable product's Child products ##

```php
<?php
// input is $_product and result is iterating child products
$childProducts = Mage::getModel('catalog/product_type_configurable')->getUsedProducts(null, $product);
```

## Get Configurable product's Children's (simple product) custom attributes ##

```php
<?php
// input is $_product and result is iterating child products
$conf = Mage::getModel('catalog/product_type_configurable')->setProduct($_product);
$col = $conf->getUsedProductCollection()->addAttributeToSelect('*')->addFilterByRequiredOptions();
foreach($col as $simple_product){
	var_dump($simple_product->getId());
}
```

## Log to custom file ##

```php
<?php Mage::log('Your Log Message', Zend_Log::INFO, 'your_log_file.log'); ?>
```

## Call Static Block ##

```php
<?php echo $this->getLayout()->createBlock('cms/block')->setBlockId('block-name')->toHtml(); ?>
```

## Add JavaScript to page ##

First approach: page.xml - you can add something like

```xml
<action method="addJs"><script>path/to/my/file.js</script></action>
```

Second approach: Find `page/html/head.phtml` in your theme and add the code directly to `page.html`.

Third approach: If you look at the stock page.html mentioned above, you'll see this line

```php
<?php echo $this->getChildHtml(); ?>
```

Normally, the getChildHtml method is used to render a specific child block. However, if called with no paramater, getChildHtml will automatically render all the child blocks. That means you can add something like

```xml
<!-- existing line --> <block type="page/html_head" name="head" as="head">
	<!-- new sub-block you're adding --> <block type="core/template" name="mytemplate" as="mytemplate" template="page/mytemplate.phtml"/>
	...
```

to `page.xml`, and then add the `mytemplate.phtml` file. Any block added to the head block will be automatically rendered. (this automatic rendering doesn't apply for all layout blocks, only for blocks where getChildHtml is called without paramaters).

## Get the current category/product/cms page ##

```php
<?php
$currentCategory = Mage::registry('current_category');
$currentProduct = Mage::registry('current_product');
$currentCmsPage = Mage::registry('cms_page');
```

## Run Magento Code Externally ##

```php
<?php
require_once('app/Mage.php'); //Path to Magento
umask(0);
Mage::app();
// Run you code here
```

## Programmatically change Magento’s core config data ##

```php
<?php
// find 'path' in table 'core_config_data' e.g. 'design/head/demonotice'
$my_change_config = new Mage_Core_Model_Config();
// turns notice on
$my_change_config->saveConfig('design/head/demonotice', "1", 'default', 0);
// turns notice off
$my_change_config->saveConfig('design/head/demonotice', "0", 'default', 0);
```

## Changing the Admin URL ##

Open up the `/app/etc/local.xml` file, locate the `<frontName>` tag, and change the ‘admin’ part it to something a lot more random, eg:

```xml
<frontName><![CDATA[supersecret-admin-name]]></frontName>
```

Clear your cache and sessions.

## Magento: Mass Exclude/Unexclude Images ##

By default, Magento will check the 'Exclude' box for you on all imported images, making them not show up as a thumbnail under the main product image on the product view.

```sql
# Mass Unexclude
UPDATE`catalog_product_entity_media_gallery_value` SET `disabled` = '0' WHERE `disabled` = '1';
# Mass Exclude
UPDATE`catalog_product_entity_media_gallery_value` SET `disabled` = '1' WHERE `disabled` = '0';
```

## getBaseUrl – Magento URL Path ##

```php
<?php
// http://example.com/
echo Mage::getBaseUrl(Mage_Core_Model_Store::URL_TYPE_WEB);
// http://example.com/js/
echo Mage::getBaseUrl(Mage_Core_Model_Store::URL_TYPE_JS);
// http://example.com/index.php/
echo Mage::getBaseUrl(Mage_Core_Model_Store::URL_TYPE_LINK);
// http://example.com/media/
echo Mage::getBaseUrl(Mage_Core_Model_Store::URL_TYPE_MEDIA);
// http://example.com/skin/
echo Mage::getBaseUrl(Mage_Core_Model_Store::URL_TYPE_SKIN);
```

## Get The Root Category In Magento ##

```php
<?php
$rootCategoryId = Mage::app()->getStore()->getRootCategoryId();
$_category = Mage::getModel('catalog/category')->load($rootCategoryId);
// You can then get all of the top level categories using:
$_subcategories = $_category->getChildrenCategories();
```

## Get The Current URL In Magento ##

```php
<?php echo Mage::helper('core/url')->getCurrentUrl(); ?>
```

## Category Navigation Listings in Magento ##

Make sure the block that you’re working is of the type catalog/navigation. If you’re editing catalog/navigation/left.phtml then you should be okay.

```php
<div id="leftnav">
	<?php $helper = $this->helper('catalog/category') ?>
	<?php $categories = $this->getStoreCategories() ?>
	<?php if (count($categories) > 0): ?>
		<ul id="leftnav-tree" class="level0">
			<?php foreach($categories as $category): ?>
				<li class="level0<?php if ($this->isCategoryActive($category)): ?> active<?php endif; ?>">
					<a href="<?php echo $helper->getCategoryUrl($category) ?>"><span><?php echo $this->escapeHtml($category->getName()) ?></span></a>
					<?php if ($this->isCategoryActive($category)): ?>
						<?php $subcategories = $category->getChildren() ?>
						<?php if (count($subcategories) > 0): ?>
							<ul id="leftnav-tree-<?= $category->getId(); ?>" class="level1">
								<?php foreach($subcategories as $subcategory): ?>
									<li class="level1<?php if ($this->isCategoryActive($subcategory)): ?> active<?php endif; ?>">
										<a href="<?= $helper->getCategoryUrl($subcategory); ?>"><?= $this->escapeHtml(trim($subcategory->getName(), '- ')); ?></a>
									</li>
								<?php endforeach; ?>
							</ul>
							<script type="text/javascript">decorateList('leftnav-tree-<?php echo $category->getId() ?>', 'recursive')</script>
						<?php endif; ?>
					<?php endif; ?>
				</li>
			<?php endforeach; ?>
		</ul>
		<script type="text/javascript">decorateList('leftnav-tree', 'recursive')</script>
	<?php endif; ?>
</div>
```

## Debug using zend ##

```php
<?php echo Zend_Debug::dump($thing_to_debug, 'debug'); ?>
```

## $_GET, $_POST & $_REQUEST Variables ##

```php
<?php
// $_GET
$productId = Mage::app()->getRequest()->getParam('product_id');
// The second parameter to getParam allows you to set a default value which is returned if the GET value isn't set
$productId = Mage::app()->getRequest()->getParam('product_id', 44);
$postData = Mage::app()->getRequest()->getPost();
// You can access individual variables like...
$productId = $postData['product_id']);
```

## Get methods of an object ##

First, use `get_class` to get the name of an object's class.

```php
<?php $class_name = get_class($object); ?>
```

Then, pass that `get_class_methods` to get a list of all the callable methods on an object

```php
<?php
$class_name = get_class($object);
$methods = get_class_methods($class_name);
foreach($methods as $method)
{
	var_dump($method);
}
```

## Is product purchasable? ##

```php
<?php if($_product->isSaleable()) { // do stuff } ?>
```

## Load Products by Category ID ##

```php
<?php
$_category = Mage::getModel('catalog/category')->load(47);
$_productCollection = $_category->getProductCollection();
if($_productCollection->count()) {
	foreach( $_productCollection as $_product ):
		echo $_product->getProductUrl();
		echo $this->getPriceHtml($_product, true);
		echo $this->htmlEscape($_product->getName());
	endforeach;
}
```

## Get associated products

In /app/design/frontend/default/site/template/catalog/product/view/type/

``` php
<?php $_helper = $this->helper('catalog/output'); ?>
<?php $_associatedProducts = $this->getAllowProducts() ?>
<?php //var_dump($_associatedProducts); ?>
<br />
<br />
<?php if (count($_associatedProducts)): ?>
	<?php foreach ($_associatedProducts as $_item): ?> 
		<a href="<?= $_item->getProductUrl(); ?>"><?= $_helper->productAttribute($_item, $_item->getName(), 'name'); ?> | <?= $_item->getName(); ?> | <?= $_item->getPrice(); ?></a>
		<br />
		<br />
	<?php endforeach; ?>
<?php endif; ?>
```

## Get An Array of Country Names/Codes in Magento ##

```php
<?php
$countryList = Mage::getResourceModel('directory/country_collection')
                    ->loadData()
                    ->toOptionArray(false);
     
    echo '<pre>';
    print_r( $countryList);
    exit('</pre>');
```

## Create a Country Drop Down in the Frontend of Magento ##

```php
<?php
$_countries = Mage::getResourceModel('directory/country_collection')
                                    ->loadData()
                                    ->toOptionArray(false) ?>
<?php if (count($_countries) > 0): ?>
    <select name="country" id="country">
        <option value="">-- Please Select --</option>
        <?php foreach($_countries as $_country): ?>
            <option value="<?= $_country['value']; ?>">
                <?= $_country['label']; ?>
            </option>
        <?php endforeach; ?>
    </select>
<?php endif; ?>
```

## Return Product Attributes ##

```php
<?php
$_product->getThisattribute();
$_product->getAttributeText('thisattribute');
$_product->getResource()->getAttribute('thisattribute')->getFrontend()->getValue($_product);
$_product->getData('thisattribute');
// The following returns the option IDs for an attribute that is a multiple-select field: 
$_product->getData('color'); // i.e. 456,499
// The following returns the attribute object, and instance of Mage_Catalog_Model_Resource_Eav_Attribute: 
$_product->getResource()->getAttribute('color'); // instance of Mage_Catalog_Model_Resource_Eav_Attribute
// The following returns an array of the text values for the attribute: 
$_product->getAttributeText('color') // Array([0]=>'red', [1]=>'green')
// The following returns the text for the attribute
if ($attr = $_product->getResource()->getAttribute('color')):
    echo $attr->getFrontend()->getValue($_product); // will display: red, green
endif;
```

## Format Price ##

```php
<?php
	$formattedPrice = Mage::helper('core')->currency($_finalPrice,true,false);
}
```


## Cart Data ##

```php
<?php
	$cart = Mage::getModel('checkout/cart')->getQuote()->getData();
	print_r($cart);
	
	$cart = Mage::helper('checkout/cart')->getCart()->getItemsCount();
	print_r($cart);
	
	$session = Mage::getSingleton('checkout/session');
	foreach ($session->getQuote()->getAllItems() as $item) {
		echo $item->getName();
		Zend_Debug::dump($item->debug());
	}
```

## Total items added in cart ##

```php
<?php
	Mage::getModel('checkout/cart')->getQuote()->getItemsCount();
	Mage::getSingleton('checkout/session')->getQuote()->getItemsCount();
```

## Total Quantity added in cart ##

```php
<?php
	Mage::getModel('checkout/cart')->getQuote()->getItemsQty();
	Mage::getSingleton('checkout/session')->getQuote()->getItemsQty();
```

## Sub Total for item added in cart ##

```php
<?php
	Mage::getModel('checkout/cart')->getQuote()->getSubtotal();
	Mage::getSingleton('checkout/session')->getQuote()->getSubtotal();
```

## Grand total for item added in cart ##

```php
<?php
	Mage::helper('checkout')->formatPrice(Mage::getModel('checkout/cart')->getQuote()->getGrandTotal());
	Mage::helper('checkout')->formatPrice(Mage::getSingleton('checkout/session')->getQuote()->getGrandTotal());
```

## Sub total of cart inkl tax without shipping ##

```php
<?php
	$quote = Mage::getSingleton('checkout/session')->getQuote();
	$items = $quote->getAllItems();
	foreach ($items as $item) {
		$priceInclVat += $item->getRowTotalInclTax();
	}
	Mage::helper('checkout')->formatPrice($priceInclVat);
```

## Get products id, name, price, quantity, etc. present in your cart ##

```php
<?php
	// $items = Mage::getModel('checkout/cart')->getQuote()->getAllItems();
	$items = Mage::getSingleton('checkout/session')->getQuote()->getAllItems();

	foreach($items as $item) {
		echo 'ID: '.$item->getProductId().'<br />';
		echo 'Name: '.$item->getName().'<br />';
		echo 'Sku: '.$item->getSku().'<br />';
		echo 'Quantity: '.$item->getQty().'<br />';
		echo 'Price: '.$item->getPrice().'<br />';
		echo "<br />";
	}
```

## Get number of items in cart and total quantity in cart ##

```php
<?php
	$totalItems = Mage::getModel('checkout/cart')->getQuote()->getItemsCount();
	$totalQuantity = Mage::getModel('checkout/cart')->getQuote()->getItemsQty();
```

## Get Simple Products of a Configurable Product ##

```php
<?php
	if($_product->getTypeId() == "configurable") {
		$ids = $_product->getTypeInstance()->getUsedProductIds();
?>
<ul>
	<?php
		foreach ($ids as $id) {
			$simpleproduct = Mage::getModel('catalog/product')->load($id);
	?>
		<li>
			<?php echo $simpleproduct->getName() . " - " . (int)Mage::getModel('cataloginventory/stock_item')->loadByProduct($simpleproduct)->getQty(); ?>
		</li>
	<?php } ?>
</ul>
<?php } ?>
```

## Reset Development Environment (delete orders, customers, products, reset ids and counters, truncate statistics) ##

```sql
SET FOREIGN_KEY_CHECKS=0;

-- Here's where we reset the orders 
TRUNCATE `sales_flat_order`;
TRUNCATE `sales_flat_creditmemo_comment`;
TRUNCATE `sales_flat_creditmemo_item`;
TRUNCATE `sales_flat_creditmemo`;
TRUNCATE `sales_flat_creditmemo_grid`;
TRUNCATE `sales_flat_invoice_comment`;
TRUNCATE `sales_flat_invoice_item`;
TRUNCATE `sales_flat_invoice`;
TRUNCATE `sales_flat_invoice_grid`;
TRUNCATE `sales_flat_quote_address_item`;
TRUNCATE `sales_flat_quote_item_option`;
TRUNCATE `sales_flat_quote`;
TRUNCATE `sales_flat_quote_address`;
TRUNCATE `sales_flat_quote_item`;
TRUNCATE `sales_flat_quote_payment`;
TRUNCATE `sales_flat_quote_shipping_rate`;
TRUNCATE `sales_flat_shipment_comment`;
TRUNCATE `sales_flat_shipment_item`;
TRUNCATE `sales_flat_shipment_track`;
TRUNCATE `sales_flat_shipment`;
TRUNCATE `sales_flat_shipment_grid`;
TRUNCATE `sales_flat_order_address`;
TRUNCATE `sales_flat_order_item`;
TRUNCATE `sales_flat_order_payment`;
TRUNCATE `sales_flat_order_status_history`;
TRUNCATE `sales_flat_order_grid`;
TRUNCATE `sendfriend_log`;
TRUNCATE `tag`;
TRUNCATE `tag_relation`;
TRUNCATE `tag_summary`;
TRUNCATE `wishlist`;
TRUNCATE `log_quote`;
TRUNCATE `report_event`;
TRUNCATE `sales_order_tax`;
TRUNCATE `sales_order_tax_item`;

ALTER TABLE `sales_flat_order` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_creditmemo_comment` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_creditmemo_item` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_creditmemo` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_creditmemo_grid` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_invoice_comment` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_invoice_item` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_invoice` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_invoice_grid` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote_address_item` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote_item_option` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote_address` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote_item` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote_payment` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_quote_shipping_rate` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_shipment_comment` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_shipment_item` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_shipment_track` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_shipment` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_shipment_grid` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_order_address` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_order_item` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_order_payment` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_order_status_history` AUTO_INCREMENT=1;
ALTER TABLE `sales_flat_order_grid` AUTO_INCREMENT=1;
ALTER TABLE `sendfriend_log` AUTO_INCREMENT=1;
ALTER TABLE `tag` AUTO_INCREMENT=1;
ALTER TABLE `tag_relation` AUTO_INCREMENT=1;
ALTER TABLE `tag_summary` AUTO_INCREMENT=1;
ALTER TABLE `wishlist` AUTO_INCREMENT=1;
ALTER TABLE `log_quote` AUTO_INCREMENT=1;
ALTER TABLE `report_event` AUTO_INCREMENT=1;
ALTER TABLE `sales_order_tax` AUTO_INCREMENT=1;
ALTER TABLE `sales_order_tax_item` AUTO_INCREMENT=1;

-- Here's where we reset the customers
TRUNCATE `customer_address_entity`;
TRUNCATE `customer_address_entity_datetime`;
TRUNCATE `customer_address_entity_decimal`;
TRUNCATE `customer_address_entity_int`;
TRUNCATE `customer_address_entity_text`;
TRUNCATE `customer_address_entity_varchar`;
TRUNCATE `customer_entity`;
TRUNCATE `customer_entity_datetime`;
TRUNCATE `customer_entity_decimal`;
TRUNCATE `customer_entity_int`;
TRUNCATE `customer_entity_text`;
TRUNCATE `customer_entity_varchar`;
TRUNCATE `log_customer`;
TRUNCATE `log_visitor`;
TRUNCATE `log_visitor_info`;

ALTER TABLE `customer_address_entity` AUTO_INCREMENT=1;
ALTER TABLE `customer_address_entity_datetime` AUTO_INCREMENT=1;
ALTER TABLE `customer_address_entity_decimal` AUTO_INCREMENT=1;
ALTER TABLE `customer_address_entity_int` AUTO_INCREMENT=1;
ALTER TABLE `customer_address_entity_text` AUTO_INCREMENT=1;
ALTER TABLE `customer_address_entity_varchar` AUTO_INCREMENT=1;
ALTER TABLE `customer_entity` AUTO_INCREMENT=1;
ALTER TABLE `customer_entity_datetime` AUTO_INCREMENT=1;
ALTER TABLE `customer_entity_decimal` AUTO_INCREMENT=1;
ALTER TABLE `customer_entity_int` AUTO_INCREMENT=1;
ALTER TABLE `customer_entity_text` AUTO_INCREMENT=1;
ALTER TABLE `customer_entity_varchar` AUTO_INCREMENT=1;
ALTER TABLE `log_customer` AUTO_INCREMENT=1;
ALTER TABLE `log_visitor` AUTO_INCREMENT=1;
ALTER TABLE `log_visitor_info` AUTO_INCREMENT=1;

-- This is to Reset all the ID counters
TRUNCATE `eav_entity_store`;
ALTER TABLE `eav_entity_store` AUTO_INCREMENT=1;

-- This is to delete all products
TRUNCATE `catalog_product_bundle_option`;
TRUNCATE `catalog_product_bundle_option_value`;
TRUNCATE `catalog_product_bundle_selection`;
TRUNCATE `catalog_product_entity_datetime`;
TRUNCATE `catalog_product_entity_decimal`;
TRUNCATE `catalog_product_entity_gallery`;
TRUNCATE `catalog_product_entity_int`;
TRUNCATE `catalog_product_entity_media_gallery`;
TRUNCATE `catalog_product_entity_media_gallery_value`;
TRUNCATE `catalog_product_entity_text`;
TRUNCATE `catalog_product_entity_tier_price`;
TRUNCATE `catalog_product_entity_varchar`;
TRUNCATE `catalog_product_link`;
TRUNCATE `catalog_product_link_attribute`;
TRUNCATE `catalog_product_link_attribute_decimal`;
TRUNCATE `catalog_product_link_attribute_int`;
TRUNCATE `catalog_product_link_attribute_varchar`;
TRUNCATE `catalog_product_link_type`;
TRUNCATE `catalog_product_option`;
TRUNCATE `catalog_product_option_price`;
TRUNCATE `catalog_product_option_title`;
TRUNCATE `catalog_product_option_type_price`;
TRUNCATE `catalog_product_option_type_title`;
TRUNCATE `catalog_product_option_type_value`;
TRUNCATE `catalog_product_super_attribute`;
TRUNCATE `catalog_product_super_attribute_label`;
TRUNCATE `catalog_product_super_attribute_pricing`;
TRUNCATE `catalog_product_super_link`;
TRUNCATE `catalog_product_enabled_index`;
TRUNCATE `catalog_product_website`;
TRUNCATE `catalog_product_entity`;

TRUNCATE `cataloginventory_stock`;
TRUNCATE `cataloginventory_stock_item`;
TRUNCATE `cataloginventory_stock_status`;

INSERT INTO `catalog_product_link_type`(`link_type_id`,`code`) VALUES (1,'relation'),(2,'bundle'),(3,'super'),(4,'up_sell'),(5,'cross_sell');
INSERT INTO `catalog_product_link_attribute`(`product_link_attribute_id`,`link_type_id`,`product_link_attribute_code`,`data_type`) VALUES (1,2,'qty','decimal'),(2,1,'position','int'),(3,4,'position','int'),(4,5,'position','int'),(6,1,'qty','decimal'),(7,3,'position','int'),(8,3,'qty','decimal');
INSERT INTO `cataloginventory_stock`(`stock_id`,`stock_name`) VALUES (1,'Default');

TRUNCATE `catalogsearch_query`;
ALTER TABLE `catalogsearch_query` AUTO_INCREMENT=1;
TRUNCATE `catalogsearch_fulltext`;
ALTER TABLE `catalogsearch_fulltext` AUTO_INCREMENT=1;

TRUNCATE `core_url_rewrite`;
TRUNCATE `adminnotification_inbox`;

SET FOREIGN_KEY_CHECKS=1;

TRUNCATE TABLE `sales_bestsellers_aggregated_daily`;
TRUNCATE TABLE `sales_bestsellers_aggregated_monthly`;
TRUNCATE TABLE `sales_bestsellers_aggregated_yearly`;

DELETE FROM report_event WHERE event_type_id IN (SELECT event_type_id FROM report_event_types WHERE event_name IN ('catalog_product_view'));
```

## set the same order increment id for all stores ##

```sql
UPDATE `eav_entity_type` SET `increment_per_store` = '0' WHERE `eav_entity_type`.`entity_type_code` = 'order';
```

## Set custom order numbers (starting number) ##

```sql
UPDATE `eav_entity_store` SET `increment_last_id` = '74395729' WHERE `entity_type_id` = 1;
```

## Update all subscribers into a customer group (e.g. 5) ##

```sql
UPDATE
	customer_entity,
	newsletter_subscriber
SET
	customer_entity.`group_id` = 5
WHERE
	customer_entity.`entity_id` = newsletter_subscriber.`customer_id`
AND
	newsletter_subscriber.`subscriber_status` = 1;
```

## Set german address format ##

```sql
INSERT INTO `directory_country_format` (`country_format_id`, `country_id`, `type`, `format`) VALUES
(1, 'DE', 'html', '{{depend prefix}}{{var prefix}} {{/depend}}{{var firstname}} {{depend middlename}}{{var middlename}} {{/depend}}{{var lastname}}{{depend suffix}} {{var suffix}}{{/depend}}<br/>\r\n{{depend company}}{{var company}}<br />{{/depend}}\r\n{{if street1}}{{var street1}}{{/if}}\r\n{{depend street2}}{{var street2}}<br />{{/depend}}\r\n{{depend street3}}{{var street3}}<br />{{/depend}}\r\n{{depend street4}}{{var street4}}<br />{{/depend}}\r\n{{if postcode}}{{var postcode}}{{/if}} {{if city}}{{var city}}{{/if}}<br/>\r\n{{var country}}<br/>\r\n{{depend telephone}}Tel.: {{var telephone}}{{/depend}}\r\n{{depend fax}}<br/>Fax: {{var fax}}{{/depend}}'),
(2, 'DE', 'pdf', '{{depend prefix}}{{var prefix}} {{/depend}}{{var firstname}} {{depend middlename}}{{var middlename}} {{/depend}}{{var lastname}}{{depend suffix}} {{var suffix}}{{/depend}}|\r\n{{depend company}}{{var company}}|{{/depend}}\r\n{{if street1}}{{var street1}} {{/if}}\r\n{{depend street2}}{{var street2}}|{{/depend}}\r\n{{depend street3}}{{var street3}}|{{/depend}}\r\n{{depend street4}}{{var street4}}|{{/depend}}\r\n{{if postcode}}{{var postcode}} {{/if}}{{if city}}{{var city}}{{/if}}}|\r\n{{var country}}|\r\n{{depend telephone}}Tel.: {{var telephone}}{{/depend}}|\r\n{{depend fax}}<br/>Fax: {{var fax}}{{/depend}}|'),
(3, 'DE', 'oneline', '{{depend prefix}}{{var prefix}} {{/depend}}{{var firstname}} {{depend middlename}}{{var middlename}} {{/depend}}{{var lastname}}{{depend suffix}} {{var suffix}}{{/depend}}, {{var street}}, {{var postcode}} {{var city}}, {{var country}}'),
(4, 'DE', 'text', '{{depend prefix}}{{var prefix}} {{/depend}}{{var firstname}} {{depend middlename}}{{var middlename}} {{/depend}}{{var lastname}}{{depend suffix}} {{var suffix}}{{/depend}}\r\n{{depend company}}{{var company}}{{/depend}}\r\n{{if street1}}{{var street1}} {{/if}}\r\n{{depend street2}}{{var street2}}{{/depend}}\r\n{{depend street3}}{{var street3}}{{/depend}}\r\n{{depend street4}}{{var street4}}{{/depend}}\r\n{{if postcode}}{{var postcode}} {{/if}}{{if city}}{{var city}}{{/if}}\r\n{{var country}}\r\nTel.: {{var telephone}}\r\n{{depend fax}}Fax: {{var fax}}{{/depend}}'),
(5, 'DE', 'js_template', '#{prefix} #{firstname} #{middlename} #{lastname} #{suffix}<br/>#{company}<br/>#{street0}<br/>#{street1}<br/>#{street2}<br/>#{street3}<br/>#{postcode} #{city}<br/>#{country_id}<br/>Tel.: #{telephone}<br/>Fax: #{fax}');
```

## Setting file permissions ##

```bash
find </path/to/magento> -type f \-exec chmod 644 {} \;
find </path/to/magento> -type d \-exec chmod 755 {} \;
```

Other recommendations in "Securing Magento File & Directory Permissions" (http://blog.nexcess.net/2010/12/06/securing-magento-file-directory-permissions/)
```bash
find </path/to/magento> \-exec chown youruser.youruser {} \;
find </path/to/magento> -type f \-exec chmod 644 {} \;
find </path/to/magento> -type d \-exec chmod 711 {} \;
find </path/to/magento> -type f -name "*.php" \-exec chmod 600 {} \;
chmod 600 </path/to/magento>/app/etc/*.xml
```

## Getting Configurable Product from Simple Product ID in Magento 1.5+ ##

```php
<?php
$simpleProductId = 465;
$parentIds = Mage::getResourceSingleton('catalog/product_type_configurable')
    ->getParentIdsByChild($simpleProductId);
$product = Mage::getModel('catalog/product')->load($parentIds[0]);
echo $product->getId(); // ID = 462 (aka, Parent of 465)
```

## Get all associated children product of a configurable product ##

```php
<?php
	/**
	* Load product by product id
	*/
	$product = Mage::getModel('catalog/product')->load(YOUR_PRODUCT_ID);
 
	/**
	* Get child products id (only ids)
	*/
	$childIds = Mage::getModel('catalog/product_type_configurable')->getChildrenIds($product->getId());
 
	/**
	* Get children products (all associated children products data)
	*/
	$childProducts = Mage::getModel('catalog/product_type_configurable')->getUsedProducts(null,$product);
```

## Get parent id of simple product associated to configurable product ##

```php
<?php
	$_product = Mage::getModel('catalog/product')->load(YOUR_SIMPLE_PRODUCT_ID);
	$parentIdArray = $_product->loadParentProductIds()->getData('parent_product_ids');
	print_r($parentIdArray);
```

## Check if customer is logged in ##

```php
<?php
	$_customer = Mage::getSingleton('customer/session')->isLoggedIn();
	if ($_customer) {}
```

## Get product image ##

```php
<?php echo $this->helper('catalog/image')->init($_product, 'image'); ?>
```

## Downsize large product images but not enlarge small images ##

```php
<?php
	$this->helper('catalog/image')
		->init($_product, 'image')
		->keepFrame(false) // avoids getting the small image in original size on a solid background color presented (can be handy not to break some layouts)
		->constrainOnly(true) // avoids getting small images bigger
		->resize(650); // sets the desired width and the height accordingly (proportional by default)
```

## No square (white background) product images ##

```html
<img src="<?php echo $this->helper('catalog/image')->init($this->getProduct(), 'thumbnail', $_image->getFile())->backgroundcolor('000', '000', '000')->resize(100); ?>" />
```

```html
<img src="<?php echo $this->helper('catalog/image')->init($this->getProduct(), 'thumbnail', $_image->getFile())->keepFrame(false)->resize(100); ?>" width="100"  ... />
```

## Show image using current skin path (PHTML) ##

```html
<img src="<?= $this->getSkinUrl('images/logo.png'); ?>" alt="logo" />
```

## Show image using current skin path (CMS) ##

```html
<img src={{skin url="images/logo.png"}}  />
```

## Show CMS block (PHTML) ##

```php
<?php echo $this->getLayout()->createBlock('cms/block')->setBlockId('my_block_identifier')->toHtml(); ?>
```

## Get Customer Shipping/Billing Address ##

```php
<?php
	$customerAddressId = Mage::getSingleton('customer/session')->getCustomer()->getDefaultShipping();
	if ($customerAddressId){
		$address = Mage::getModel('customer/address')->load($customerAddressId);
	}
```

## Get Product image path ##

```php
<?php
	$productId = 1;
	$product = Mage::getModel('catalog/product')->load($productId);
	$path = Mage::helper('catalog/image')->init($product, 'image')->resize(75, 75);
```

## Get product URL ##

```php
<?php
	$productId = 1;
	$product = Mage::getModel('catalog/product')->load($productId);
	$path = Mage::getUrl().$product->getUrlPath();
```

## Get Category URL ##

```php
<?php echo Mage::getModel('catalog/category')->load($categoryId)->getUrl(); ?>
```

## Get product stock quantity ##

```php
<?php
	$id = 52;
	$_product = Mage::getModel('catalog/product')->load($id);

	// or load it by SKU
	// $sku = "microsoftnatural";
	// $_product = Mage::getModel('catalog/product')->loadByAttribute('sku', $sku);

	$stock = Mage::getModel('cataloginventory/stock_item')->loadByProduct($_product);
	
	print_r($stock->getData());
	
	echo $stock->getQty();
	echo $stock->getMinQty();
	echo $stock->getMinSaleQty();
```

## Get actual price and special price of a product ##

```php
<?php
	$_productId = 52;
	$_product = Mage::getModel('catalog/product')->load($_productId);

	// without currency sign
	$_actualPrice = number_format($_product->getPrice(), 2);
	// with currency sign
	$_formattedActualPrice = Mage::helper('core')->currency(number_format($_product->getPrice(), 2),true,false);

	// without currency sign
	$_specialPrice = $_product->getFinalPrice();
	// with currency sign
	$_formattedSpecialPrice = Mage::helper('core')->currency(number_format($_product->getFinalPrice(), 2),true,false);
```

## Get Currency Symbol ##

```php
<?php echo Mage::app()->getLocale()->currency(Mage::app()->getStore()->getCurrentCurrencyCode())->getSymbol(); ?>
```

## Get Currency Code ##

```php
<?= Mage::app()->getStore()->getCurrentCurrencyCode(); ?>
```

## Track Visitor’s Information ##

```php
<?php
	$visitorData = Mage::getSingleton('core/session')->getVisitorData();
	print_r($visitorData);
	
	// Array
	// (
	// [] =>
	// [server_addr] => 167772437
	// [remote_addr] => 167772437
	// [http_secure] =>
	// [http_host] => 127.0.0.1
	// [http_user_agent] => Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.10 (KHTML, like Gecko) Chrome/8.0.552.237 Safari/534.10
	// [http_accept_language] => en-US,en;q=0.8
	// [http_accept_charset] => ISO-8859-1,utf-8;q=0.7,*;q=0.3
	// [request_uri] => /magento/index.php/catalog/category/view/id/22
	// [session_id] => 13qm5u80238vb15lupqcac97r5
	// [http_referer] => http://127.0.0.1/magento/
	// [first_visit_at] => 2011-01-17 11:42:23
	// [is_new_visitor] =>
	// [last_visit_at] => 2011-01-17 11:58:38
	// [visitor_id] => 41
	// [last_url_id] => 139
	// )
	
	// user's ip address (visitor's ip address)
	$remoteAddr = Mage::helper('core/http')->getRemoteAddr(true);

	// server's ip address (where the current script is)
	$serverAddr = Mage::helper('core/http')->getServerAddr(true);
```

## Get / filter all products by attribute value ##

```php
<?php
	/**
	* Get all products related to any particular brand
	* Let us suppose that we are fetching the products related to 'Samsung' brand
	* Let us suppose the Manufacturer ID of Samsung = 3
	*/
	
	$manufacturerId = 3;
	$attributeCode = 'manufacturer';
	
	$products = Mage::getModel('catalog/product')->getCollection()->addAttributeToFilter($attributeCode, $manufacturerId);
	
	// print all products
	print_r($products->getItems());
```

## Check if current page is homepage ##

```php
<?php
	if($this->getIsHomePage()) {
		// Homepage
	} else {
		// not on Homepage
	}

	// alternative method
	$routeName = Mage::app()->getRequest()->getRouteName();
	$identifier = Mage::getSingleton('cms/page')->getIdentifier();
	if($routeName == 'cms' && $identifier == 'home') {
		// Homepage
	} else {
		// not on Homepage
	}
```

## Convert Price from Current Currency to Base Currency and vice-versa ##

```php
<?php
	$baseCurrencyCode = Mage::app()->getStore()->getBaseCurrencyCode();
	$currentCurrencyCode = Mage::app()->getStore()->getCurrentCurrencyCode();
	$price = 100;
	
	// convert price from current currency to base currency
	$priceOne = Mage::helper('directory')->currencyConvert($price, $currentCurrencyCode, $baseCurrencyCode); 
	
	// convert price from base currency to current currency
	$priceTwo = Mage::helper('directory')->currencyConvert($price, $baseCurrencyCode, $currentCurrencyCode);
```

## Changing price from any one currency to another ##

```php
<?php
	$from = 'USD';
	$to = 'NPR';
	$price = 10;

	$newPrice = Mage::helper('directory')->currencyConvert($price, $from, $to);
```

## Get Currency Rates ##

```php
<?php
	/**
	 * Get the base currency
	 */
	$baseCurrencyCode = Mage::app()->getBaseCurrencyCode();

	/**
	 * Get all allowed currencies
	 * returns array of allowed currency codes
	 */
	$allowedCurrencies = Mage::getModel('directory/currency')->getConfigAllowCurrencies();

	/**
	 * Get the currency rates
	 * returns array with key as currency code and value as currency rate
	 */
	$currencyRates = Mage::getModel('directory/currency')->getCurrencyRates($baseCurrencyCode, array_values($allowedCurrencies));

	$allowedCurrencies = Mage::getModel('directory/currency')->getConfigAllowCurrencies();

	/**
	 * Get currency rates for Nepalese Currency
	 */
	$currencyRates = Mage::getModel('directory/currency')->getCurrencyRates('NPR', array_values($allowedCurrencies));
```

## Get all categories ##

```php
<?php
	$categories = Mage::getModel('catalog/category')->getCollection()->addAttributeToSelect('*');
?>
```

## Get all active categories ##

```php
<?php
	$categories = Mage::getModel('catalog/category')->getCollection()->addAttributeToSelect('*')->addIsActiveFilter();
```

## Get active categories of any particular level ##

```php
<?php
	$categories = Mage::getModel('catalog/category')->getCollection()->addAttributeToSelect('*')->addIsActiveFilter()->addLevelFilter(1)->addOrderField('name');
```

## Get store specific categories ##

```php
<?php
	$helper = Mage::helper('catalog/category');
	
	// sorted by name, fetched as collection
	$categoriesCollection = $helper->getStoreCategories('name', true, false);
	
	// sorted by name, fetched as array
	$categoriesArray = $helper->getStoreCategories('name', false, false);
```

## Get product in stock quantity ##

```php
<?php
	$_product = Mage::getModel('catalog/product')->load($product_id);
	$qty = Mage::getModel('cataloginventory/stock_item')->loadByProduct($_product)->getQty();
```

## Use magento "outside" magento ##

```php
<?php
	require_once 'app/Mage.php';
	Mage::app($yourStoreCode);
	echo Mage::getStoreConfig('general/store_information/name');
```

## Hijack session outside magento ##

```php
<?php
	require_once 'app/Mage.php';
	Mage::app($yourStoreCode);
	Mage::getSingleton('core/session', array('name'=>'frontend'))->setSessionId($_COOKIE['frontend']);
	echo "cart_id=".Mage::helper('checkout/cart')->getCart()->getQuote()->getId();
```

## Filter collection or get configurable products: ##

```php
<?php
	$configurable_products = Mage::getModel('catalog/product')
		->getCollection()
		->addAttributeToSelect('*')
		->addAttributeToFiler('type_id',array('eq'=>'configurable'))
		->load();
```

## Round price ##

```php
<?php
	echo Mage::getModel('sales/order')->formatPricePrecision($_product->getFinalPrice(), 3);
```

## Get a list of bestsellers ##

```php
<?php
$collection = Mage::getResourceModel('sales/report_bestsellers_collection')
    ->setModel('catalog/product')
    ->addStoreFilter(Mage::app()->getStore()->getId()) //if you want the bestsellers for a specific store view. if you want global values remove this
    ->setPageSize(5)//set the number of products you want
    ->setCurPage(1);
foreach ($collection as $_product){
    $realProduct = Mage::getModel('catalog/product')->load($_product->getProductId());
    //do something with $realProduct;
}
```

## Get product categories with name ##

```php
<?php 
$categoryCollection = Mage::getResourceModel('catalog/category_collection')
	->joinField('product_id',
		'catalog/category_product',
		'product_id',
		'category_id = entity_id',
		null)
	->addAttributeToSelect('name')
	->addFieldToFilter('product_id', (int)$_product->getId());
?>
```

## Add category names in product view page ##

```php
<?php 
	$categoryIds = $_product->getCategoryIds();
	foreach ($categoryIds as $categoryId){
		$tmpId = $categoryId;
		$categories = array();
		while($tmpId != Mage::app()->getStore()->getRootCategoryId()) {
			$category = Mage::getModel('catalog/category')->setStoreId(Mage::app()->getStore()->getId())->load($tmpId);
			$categories[] = $category;
			$tmpId = $category->getParentId();
		};
		for ($i = count($categories) - 1; $i>=0;$i--){
			echo '<a href="'.echo $categories[$i]->getUrl().'">'.echo $categories[$i]->getName().'</a>';
			if ($i>0){
				echo "-&gt;"<!-- this is the tree separator. change to whatever you like-->
			}
		}
		//break;//uncomment this if you want only one category tree to appear.
	};
```

## Different toolbar in product list / grid ##

```php
<?php echo $this->getToolbarBlock()->setTemplate('catalog/product/list/toolbar_bottom.phtml')->toHtml(); ?>
```

## remotely trigger Varien.Tabs Class or EasyTabs ##

add this to Varien.Tabs.prototype
```js
Varien.Tabs.prototype = {
remoteTabs: function(b) {
var controlledLink = $$("#"+b+" a")[0];
this.showContent(controlledLink);
}
}
var csTablist = new Varien.Tabs('.tabs');
```
to fire the link remotely you can call
```js
csTablist.remoteTabs('product_tabs_email');
```
where product_tabs_email is the name of the li that you want to open.

## Getting Configurable Attributes (Super Attributes) of a Configurable Product ##

```php
<?php 
	/**
	* Load Product you want to get Super attributes of
	*/
	$product=Mage::getModel("catalog/product")->load(52520);

	/**
	* Get Configurable Type Product Instace and get Configurable attributes collection
	*/
	$configurableAttributeCollection=$product->getTypeInstance()->getConfigurableAttributes();

	/**
	* Use the collection to get the desired values of attribute
	*/
	foreach($configurableAttributeCollection as $attribute){
		echo "Attr-Code:".$attribute->getProductAttribute()->getAttributeCode()."<br/>";
		echo "Attr-Label:".$attribute->getProductAttribute()->getFrontend()->getLabel()."<br/>";
		echo "Attr-Id:".$attribute->getProductAttribute()->getId()."<br/>";
		var_dump($_attribute->debug()); // returns the set of values you can use the get magic method on
	}
```

## Get order information on success.phtml ##

```php
<?php
$_customerId = Mage::getSingleton('customer/session')->getCustomerId();
$lastOrderId = Mage::getSingleton('checkout/session')->getLastOrderId();
$order = Mage::getSingleton('sales/order');
$order->load($lastOrderId);
$_totalData =$order->getData();
$_grand = $_totalData['grand_total'];
```

## Order date with correct GMT offset on Grid

```
$this->addColumn('order_created_at', array(
    'header'    => Mage::helper('customer')->__('Datum'),
    'index'     => 'order_created_at',
    'gmtoffset' => true,
    'type'      => 'datetime',  
));
```

## Check if product has an image

```php
<?php
$has_real_image_set = ($_product->getSmallImage() != null && $_product->getSmallImage() != "no_selection");
if ($has_real_image_set) echo "has image";
else echo "no image";
```

## Add table column via sql installer ##

```php
<?php
$installer = $this;
$installer->startSetup();
$installer->getConnection()
    ->addColumn($installer->getTable('module/entity'),
        'column_name',
        array(
            'type' => Varien_Db_Ddl_Table::TYPE_INTEGER,
            'length' => 1,
            'nullable' => false,
            'default' => 0,
            'comment' => 'Breifly describe the new column'
        )
    );

$installer->endSetup();
```

## Get media gallery by product

```php
<?php
...
$product = Mage::getModel('catalog/product')->getCollection()
    ->addAttributeToFilter('entity_id', array('eq' => '1'))
    ->getFirstItem();

$galleryAttribute = $product->getResource()->getAttribute('media_gallery');
$gallerySomeShit = Mage::getModel('catalog/product_attribute_backend_media')->setAttribute($galleryAttribute);
$gallery = Mage::getResourceModel('catalog/product_attribute_backend_media')->loadGallery($product, $gallerySomeShit);
```

## Add product attribute with sql installer

config.xml

```xml
<config>
    <global>
        ...
        <resources>
            <microsite_module_setup>
                <setup>
                    <module>MODA_Product</module>
                    <class>Mage_Catalog_Model_Resource_Eav_Mysql4_Setup</class>
                </setup>
            </microsite_module_setup>
        </resources>
        ...
    </global>
</config>
```
Setup file
```php
<?php
$installer->addAttribute('catalog_product', 'some_attribute', array(
    'group'             => 'General',
    'label'             => 'Product Page Module',
    'note'              => '',
    'type'              => 'int',    //backend_type
    'input'             => 'select', //frontend_input
    'frontend_class'    => '',
    'source'            => 'moda_product/attribute_source_microsite', // optional
    'backend'           => '',
    'frontend'          => '',
    'global'            => Mage_Catalog_Model_Resource_Eav_Attribute::SCOPE_WEBSITE,
    'required'          => false,
    'visible_on_front'  => true,
    'apply_to'          => 'simple,configurable',
    'is_configurable'   => false,
    'used_in_product_listing'   => false,
    'sort_order'        => 20,
));
```

## Get all methods for product
```php
<?php
Zend_Debug::dump(get_class_methods(get_class($product)))
```

## Magento notification ##
error:
```php
<?php
Mage::getSingleton('core/session')->addError('Custom error here');
```
warning:
```php
<?php
Mage::getSingleton('core/session')->addWarning('Custom warning here');
```
notice:
```php
<?php
Mage::getSingleton('core/session')->addNotice('Custom notice here');
```
success:
```php
<?php
Mage::getSingleton('core/session')->addSuccess('Custom success here');
```
or in admin controller:
```php
<?php
$this->_getSession()->addSuccess($this->__('text'));
$this->_getSession()->addError($this->__('text'));
```

## Redirect
in controller:
```php
<?php
$this->getResponse()->setRedirect("/")->sendResponse();
```
otherwise: 
```php
<?php
Mage::app()->getFrontController()->getResponse()->setRedirect($url);
```

## Pass data from controller to view
```php
<?php
$this->getLayout()->getBlock("cityinfo_index")->setData("data", $data);
```
in .phtml:
```php
<?php echo $this->getData("data"); ?>
```

## Set meta data in controller
```php
<?php
$head = $this->getLayout()->getBlock('head');
$head->setTitle($data["city_info"]->meta_title);
$head->setKeywords($data["city_info"]->meta_keywords);
$head->setDescription($data["city_info"]->meta_description);
```

## Get all stores
```php
<?php
$allStores = Mage::app()->getStores();
foreach ($allStores as $id => $value)
{
    $storeCode = Mage::app()->getStore($id)->getCode();
    $storeName = Mage::app()->getStore($id)->getName();
    $storeId = Mage::app()->getStore($id)->getId();
}
```

## Add rewrite url
```php
<?php
$rewrite = Mage::getModel('core/url_rewrite');
$rewrite->setStoreId($storeId)
    ->setIdPath('path_1')
    ->setRequestPath('mego-page.html')
    ->setTargetPath('cityinfo/index/index/city/Киев')
    ->setIsSystem(false)
    ->save();
```

## Delete "add" button from admin grid
```php
<?php
parent::__construct();
$this->_removeButton('add'); // call after parent construct
```

## Sort collection
```php
<?php
$cities = Mage::getModel("cityinfo/cityinfo")->getCollection()->setOrder("city", "ASC");
````

## insert template in cms page
```html
{{block type="core/template" name="my.block.name" template="myfolder/newfile.phtml"}}
```

## Get url for deleting product from list of comparing
```php
<?php
$compareRemoveUrl = $this->helper('catalog/product_compare')->getRemoveUrl($_product);
```

## Get add to compare link
```php
<?php
$_product = Mage::getModel('catalog/product')->load($id);
echo Mage::helper('catalog/product_compare')->getAddUrl($_product);
```

## Check if product are in compare list
```php
<?php
$compared = false;

$collection = $this->helper('catalog/product_compare')->getItemCollection();
foreach($collection as $comparing_product) {
    if ($comparing_product->getId() === $_product->getId()) {
        $compared = true;
    }
}
```

## get product url from review block ##
```php
<?php
$this->getProduct()->getUrlInStore()
```

## Filter by AND
```php
<?php
->addFieldToFilter(array(
    array('attribute'=>'action','eq'=> 1),
))
->addFieldToFilter(array(
    array('attribute'=>'status','eq'=> 1),
))
```

## Filter by OR
```php
<?php
->addFieldToFilter(
    array(
        array('attribute'=>'my_field1','eq'=>'my_value1'),
        array('attribute'=>'my_field2', 'eq'=>'my_value2')
    )
);

## Add new attribute option value
```php
<?php
$arg_attribute = 'youtube_video'; // name of attribute
$arg_value = 'value to be added'; // option value

$attr_model = Mage::getModel('catalog/resource_eav_attribute');
$attr = $attr_model->loadByCode('catalog_product', $arg_attribute);
$attr_id = $attr->getAttributeId();

$option['attribute_id'] = $attr_id;
$option['value']['youtube_video'][0] = $arg_value;

$setup = new Mage_Eav_Model_Entity_Setup('core_setup');
$setup->addAttributeOption($option);
```

## Show CMS block by layout
```xml
<block type="cms/block" name="home-page-block">
    <action method="setBlockId"><block_id>home-page-block</block_id></action>
</block>
```

## Create block
```php
<?php
Mage::app()->getLayout()->createBlock('Mage_Core_Block_Text');
```

## Get image original URL
```php
<?php
$productMediaConfig = Mage::getModel('catalog/product_media_config');
$originalImage = $productMediaConfig->getMediaUrl($_product->getImage());
```

## Get image sizes
```php
<?php
$bigImage = $this->helper('catalog/image')->init($_product, 'image');
$bigImage->getOriginalWidth();
$bigImage->getOriginalHeight();
```

## Get login URL
```php
<?php
Mage::helper('customer')->getLoginUrl()
```

## Get logout URL
```php
<?php
Mage::helper('customer')->getLogoutUrl()
```

## Get attribute type (select, multiselect)
```php
<?php
$attribute_details = Mage::getSingleton("eav/config")->getAttribute('catalog_product', $name);
$attribute = $attribute_details->getData();
$attributeType = $attribute->getFrontendInput();
```

## Get attribute values
```php
<?php
$attribute_details = Mage::getSingleton("eav/config")->getAttribute('catalog_product', $name);
$attributeValues = [];
foreach ($attribute_details->getSource()->getAllOptions(true, true) as $option){
	$attributeValues[$option['value']] = $option['label'];
}
```

## Get product attribute set
```php
<?php
$attributeSet = Mage::getModel('eav/entity_attribute_set')->load($_product->getAttributeSetId())->getAttributeSetName();
```

## Update product position in specific category
```php
<?php
Mage::app()->setCurrentStore(Mage::getModel('core/store')->load(Mage_Core_Model_App::ADMIN_STORE_ID));
$categoryId = 22; //replace with your category id
$newPosition = 100; //replace with your new position
$category = Mage::getModel('catalog/category')->setStoreId(Mage_Core_Model_App::ADMIN_STORE_ID)->load($categoryId);
$products = $category->getProductsPosition();
foreach ($products as $id=>$value){
    $products[$id] = $newPosition;
}
$category->setPostedProducts($products);
$category->save();
```