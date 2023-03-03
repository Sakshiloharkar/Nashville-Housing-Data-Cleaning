# Nashville-Housing-Data-Cleaning

SELECT * FROM Data;


# 1. Standardize date format

ALTER TABLE Data
ADD SalesDateCon Date;

UPDATE Data
SET SalesDateCon = CONVERT(DATE,SaleDate);

SELECT * FROM Data;


# 2. Populate Property address 

UPDATE a
SET PropertyAddress  = ISNULL(a.PropertyAddress,b.PropertyAddress)
FROM Data a
JOIN Data b ON a.ParcelID=b.ParcelID AND 
a.[UniqueID]<>b.[UniqueID]
WHERE a.PropertyAddress IS NULL


# 3. Breaking out address into Individual columns (Address, City, State)

ALTER TABLE Data
ADD PropertySplitAddress nvarchar(255);

UPDATE Data 
SET PropertySplitAddress = SUBSTRING(PropertyAddress,1,CHARINDEX(',',PropertyAddress)-1);

ALTER TABLE Data
ADD PropertySplitCity nvarchar(255);

UPDATE Data 
SET PropertySplitCity = SUBSTRING(PropertyAddress,CHARINDEX(',',PropertyAddress)+1,LEN(PropertyAddress));


# 4. Populate Owner address

ALTER TABLE Data
ADD OwnerSplitAddress nvarchar(255);

UPDATE Data
SET OwnerSplitAddress=PARSENAME(REPLACE(OwnerAddress,',','.'),3);

ALTER TABLE Data
ADD OwnerSplitCity nvarchar(255);

UPDATE Data
SET OwnerSplitCity=PARSENAME(REPLACE(OwnerAddress,',','.'),2);

ALTER TABLE Data
ADD OwnerSplitState nvarchar(255);

UPDATE Data
SET OwnerSplitState=PARSENAME(REPLACE(OwnerAddress,',','.'),1);

SELECT * FROM Data;


# 5. Change Y & N to yes & no in sold s vacant.

ALTER TABLE Data
ADD SoldAsVacantUpdated nvarchar(255);

UPDATE Data
SET SoldAsVacantUpdated = (CASE WHEN SoldAsVacant='Y' THEN 'Yes'
	 WHEN SoldAsVacant='N' THEN 'No'
	 ELSE SoldAsVacant 
	 END);

SELECT DISTINCT SoldAsVacantUpdated FROM Data


# 6. Remove duplicates

WITH rownumcte AS(
SELECT *,ROW_NUMBER() OVER(PARTITION BY ParcelId,PropertyAddress,SaleDate,SalePrice,LegalReference 
					ORDER BY UniqueID) AS rownum
FROM Data)

DELETE FROM rownumcte
WHERE rownum>1
--order by PropertyAddress

SELECT * FROM Data


# 7. Delete Unused Columns

ALTER TABLE Data
DROP COLUMN PropertyAddress,TaxDistrict,OwnerAddress

ALTER TABLE Data
DROP COLUMN SaleDate;
