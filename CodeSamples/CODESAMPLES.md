# Code Samples
- [Median of Two Sorted Arrays - C++](#median-of-two-sorted-arrays---c)
- [Tower Defense Events - C#](#modular-tower-defense-events---c)
- [Ray-Plane Intersection](#finding-ray-plane-intersection)
- [Game Jam Game](#3-bullets)
- [2D Platformer](#2d-platformer-demo)
# Median of Two Sorted Arrays - C++
Code for finding the median of two sorted arrays in O Log time. Begins at findMedianSortedArrays method at bottom of block.
``` C++
class Solution {
public:
    // Holds relevant information about how close a number is to being the median.
    class MiddleElement {
        public:
            // Count of numbers less than this number in the two arrays.
            int numLessThan = INT_MIN / 3;
            // Count of numbers greater than this number in the two arrays.
            int numGreaterThan = INT_MAX / 3;
            // Pointer to the value of this number.
            int* pValue; 
            int getDifferenceOfLessGreater() { return abs(numGreaterThan-numLessThan); }    
    };

    struct MedianSearchResult { bool found = false; int median = INT_MIN; };

    //Tracks the two values closest to the median to use for certain edge cases.
    struct MiddlePair { MiddleElement element1; MiddleElement element2; };

    // Updates the middle pair if the new value is closer to the median.
    void updateMiddlePair(int numLessThan, int numGreaterThan, int* newValue, MiddlePair* middlePair) {
        // New value being evaluated points to same place as an already existing middlePair element, we don't add duplicates.
        if (middlePair->element1.pValue == newValue || middlePair->element2.pValue == newValue)
            return;
        // Smaller diff means the closer number is to median.
        int newDiff = abs(numGreaterThan - numLessThan);
        // replace largest diff
        MiddleElement* elementToChange = &middlePair->element1;
        if (middlePair->element2.getDifferenceOfLessGreater() > middlePair->element1.getDifferenceOfLessGreater())
            elementToChange = &middlePair->element2;
        
        // Update elements members to new number.
        if (newDiff < elementToChange->getDifferenceOfLessGreater()) {
            elementToChange->numLessThan = numLessThan;
            elementToChange->numGreaterThan = numGreaterThan;
            elementToChange->pValue = newValue;
        }
    }
    // Updates indice information when we search the lower half of array.
    void updateIndiceRangeLowerHalf(int* searchIndex, int* searchStart, int* searchEnd) {
        *searchEnd = *searchIndex;
        int indexOffset = (*searchEnd - *searchStart) / 2;
        *searchIndex = *searchIndex - indexOffset - 1;
    }
    // Updates indice information when we search the upper half of array.
    void updateIndiceRangeUpperHalf(int* searchIndex, int* searchStart, int* searchEnd) {
        *searchStart = *searchIndex;
        int indexOffset = (*searchEnd - *searchIndex) / 2;
        *searchIndex = *searchIndex + indexOffset;
    }

    // Finds the count of numbers in an array that are smaller than the given number.
    // @param[in] num Number used to count numbers larger than.
    // @param[in] arrayToSearch Array to count
    // @param[in] searchStart Beginning index of current step in binary search
    // @param[in] searchEnd Used to know where the end is in current step in binary search. Equal to 1 + maximum index of current step.
    // @param[in] searchIndex Beginning index of search, should start as arrayToSearch.size()/2
    int getCountOfNumbersSmallerInArray(int num, std::vector<int>* arrayToSearch, int searchStart, int searchEnd, int searchIndex) {
        // If our current index is outside the bounds of the array, the number is not in this array.
        if (searchIndex < 0)
            return 0;
        else if (searchIndex >= arrayToSearch->size())
            return static_cast<int>(arrayToSearch->size());

        int currentNum = arrayToSearch->at(searchIndex);

        // If we've reached the closest we will get to the number, count the amount of smaller numbers, which
        // is the remaining size of the array past this number.
        if (searchEnd - searchStart == 1) {
            int count = searchIndex;
            // Closest number is included in count if its less than number.
            if (currentNum < num)
                count += 1;
            return count;
        }
        // If the number we're searching for is greater than the current number,
        // recursively search upper half of the array, otherwise search the lower half.
        if (num > currentNum) {
            updateIndiceRangeUpperHalf(&searchIndex, &searchStart, &searchEnd);
            return getCountOfNumbersSmallerInArray(num, arrayToSearch, searchStart, searchEnd, searchIndex);
        }
        // If the number we're searching for is less than the current number,
        // recursively search lower half of the array, otherwise search the upper half.
        else {
            updateIndiceRangeLowerHalf(&searchIndex, &searchStart, &searchEnd);
            return getCountOfNumbersSmallerInArray(num, arrayToSearch, searchStart, searchEnd, searchIndex);
        }
        return -1;
    }

    // Finds the count of numbers in an array that are larger than the given number.
    // @param[in] num Number used to count numbers larger than.
    // @param[in] arrayToSearch Array to count
    // @param[in] searchStart Beginning index of current step in binary search
    // @param[in] searchEnd Used to know where the end is in current step in binary search. Equal to 1 + maximum index of current step.
    // @param[in] searchIndex Beginning index of search, should start as arrayToSearch.size()/2
    int getCountOfNumbersLargerInArray(int num, std::vector<int>* arrayToSearch, int searchStart, int searchEnd, int searchIndex) {

        // If our search index is below 0, the entire array is larger than the number.
        if (searchIndex < 0)
            return static_cast<int>(arrayToSearch->size());
        // If our search index is greater than or equal to the size of the array, no numbers are larger.
        else if (searchIndex >= arrayToSearch->size())
            return 0;

        int currentNum = arrayToSearch->at(searchIndex);
        // If we've reached the closest we will get to the number, count the amount of larger numbers, which
        // is the remaining size of the array past this number.
        if (searchEnd - searchStart == 1) {
            int count = static_cast<int>(arrayToSearch->size()) - searchIndex;
            // Closest number is excluded from count if its less or equal to the number.
            if (currentNum <= num)
                count -= 1;
            return count;
        }
        // If the number we're searching for is less than the current number,
        // recursively search lower half of the array, otherwise search the upper half.
        if (num < currentNum) {
            updateIndiceRangeLowerHalf(&searchIndex, &searchStart, &searchEnd);
            //searchIndex -= 1;
            return getCountOfNumbersLargerInArray(num, arrayToSearch, searchStart, searchEnd, searchIndex);
        }
        else {
            updateIndiceRangeUpperHalf(&searchIndex, &searchStart, &searchEnd);
            return getCountOfNumbersLargerInArray(num, arrayToSearch, searchStart, searchEnd, searchIndex);
        }
        return -1;
    }

    // Searches one of two arrays in a pair of sorted arrays to see if it contains the median value of the two arrays.
    // @param[in] arrayToSearch Array to be searched if it contains the median
    // @param[in] searchStart Beginning index of current step in binary search
    // @param[in] searchEnd Used to know where the end is in current step in binary search. Equal to 1 + maximum index of current step.
    // @param[in] searchIndex Beginning index of search, should start as arrayToSearch.size()/2
    // @param[in] otherArray The other array in the array pair, used to find number of smaller and larger values to verify median.
    // @param[in] middlePair Tracks the closest values to the median in edge cases such as an even number of total elements.
    MedianSearchResult findMedianInArray(std::vector<int>* arrayToSearch, int searchStart, int searchEnd, int searchIndex, std::vector<int>* otherArray, MiddlePair* middlePair){
        // Don't do search if array is empty.
        if (arrayToSearch->size() == 0)
            return MedianSearchResult{};

        int* indexValue = &arrayToSearch->at(searchIndex);

        // Finds the amount of numbers that are smaller and larger than the current number
        int otherArrayNumSmallerThan = getCountOfNumbersSmallerInArray(*indexValue, otherArray, 0, otherArray->size(), otherArray->size() / 2);
        int otherArrayNumLargerThan = getCountOfNumbersLargerInArray(*indexValue, otherArray, 0, otherArray->size(), otherArray->size() / 2);
        int searchArrayNumSmallerThan = getCountOfNumbersSmallerInArray(*indexValue, arrayToSearch, 0, arrayToSearch->size(), arrayToSearch->size() / 2);
        int searchArrayNumLargerThan = getCountOfNumbersLargerInArray(*indexValue, arrayToSearch, 0, arrayToSearch->size(), arrayToSearch->size() / 2);
        
        int lessThanSum = searchArrayNumSmallerThan + otherArrayNumSmallerThan;
        int greaterThanSum = searchArrayNumLargerThan + otherArrayNumLargerThan;
        // Updates our middlePair containing information about the two closest numbers to the median.
        updateMiddlePair(lessThanSum,greaterThanSum, indexValue, middlePair);
        // If the amount of numbers that are smaller than our current number is equal to the amount of numbers that are 
        // larger, this is the median. Otherwise, perform another step of the binary search.
        if (lessThanSum == greaterThanSum) {
            return MedianSearchResult{ true, *indexValue };
            std::cout << *indexValue << "\n";
        }
        else {
            // Our current number is too high, search the lower half of the array.
            if (lessThanSum > greaterThanSum) {
                // If our number is too high and we're at the first index in the array, the median doesn't exist in this array
                if (searchIndex == 0)
                    return MedianSearchResult{};

                updateIndiceRangeLowerHalf(&searchIndex, &searchStart, &searchEnd);
                return findMedianInArray(arrayToSearch, searchStart, searchEnd, searchIndex, otherArray, middlePair);
            }
            // Our current number is too low, search the upper half of the array.
            else {
                // If there are no more numbers to search in our range, the median doesn't exist in this array.
                if (searchEnd - searchStart == 1) {
                    int arrayLength = static_cast<int>(arrayToSearch->size());
                    return MedianSearchResult{};
                }
                updateIndiceRangeUpperHalf(&searchIndex, &searchStart, &searchEnd);
                return findMedianInArray(arrayToSearch, searchStart, searchEnd, searchIndex, otherArray, middlePair);
            }
        }
        return MedianSearchResult{};
    }
    // Returns the median value of two sorted arrays
    double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
        MiddlePair middlePair = MiddlePair();
        MedianSearchResult medianResult = MedianSearchResult();
        medianResult = findMedianInArray(&nums1, 0, nums1.size(), nums1.size() / 2, &nums2, &middlePair);
        // Median not in first array, check second array.
        if (!medianResult.found) {
            medianResult = findMedianInArray(&nums2, 0, nums2.size(), nums2.size() / 2, &nums1, &middlePair);
        }
        if (medianResult.found) {
            std::cout << "median: " << medianResult.median << "\n";
            return medianResult.median;
        }
        // If we didn't find a median in either array, its determined by the middlePair.
        if (!medianResult.found) {
            // If there is only one valid MiddlePair element, that is the median.
            if (middlePair.element2.pValue == NULL) {
                double median = *middlePair.element1.pValue;
                std::cout << "median: " << median << "\n";
                return median;
            }
            // The median is the mean of the two middlePair values.
            else {
                // Invalid middle pair
                if (*middlePair.element1.pValue == INT_MAX || *middlePair.element2.pValue == INT_MAX) {
                    std::cout << "Incorrect middle pair error!" << "\n";
                    return -1;
                }
                return (*middlePair.element1.pValue + *middlePair.element2.pValue) / 2.0;
            }
        }
        return -1;
    }
};
```

Solution available on [Github](https://github.com/ZackDG/MedianOfTwoSortedArrays/blob/main/MedianOfArrays/MedianOfArrays.cpp) and [Leet Code](https://leetcode.com/submissions/detail/1190908838/).

# Modular Tower Defense Events - C\#
Towers have event callbacks that invoke any necessary methods for any module that needs it. This allows modules to have their own game logic contained within themselves and only requiring a small amount of code to touch other systems.
Certain specific modules can have their own method calls, or it could be made more general with base classes and inheritance.

``` C#
    /// <summary>
    /// Event callback invoked when the tower's base object creates a new TDObject. Sends necessary information
    /// to any triggers that will use it. (Used for triggers such as On Hit).
    /// </summary>
    /// <param name="obj">Tower's Base Object</param>
    void BaseObjectCreatedCallback(GameObject obj){
        foreach(GameObject module in AdditionalModules){
            if(module.gameObject.GetComponent<Trigger>() != null){
                Trigger triggerModule = module.gameObject.GetComponent<Trigger>();
                if(triggerModule is TriggerOnHit){
                    (triggerModule as TriggerOnHit).SetTriggeringObject(obj);
                }
                else if(triggerModule is TriggerOnBleedDeath){
                    (triggerModule as TriggerOnBleedDeath).SetTriggeringObject(obj);
                }
            }
        }    
    }
    /// <summary>
    /// Callback when tower attacks to begin any triggers that start on attack.
    /// </summary>
    void BaseAttackCallback(){
        if(TowerAnimator != null){
            TowerAnimator.OnAttack();
        }
        audioSource.PlayDelayed(AttackAudioDelaySeconds);
        foreach(GameObject module in AdditionalModules){
            if(module.gameObject.GetComponent<Trigger>() != null){
                Trigger triggerModule = module.gameObject.GetComponent<Trigger>();   
                if(triggerModule is TriggerOnAttack){
                    (triggerModule as TriggerOnAttack).OnAttack();
                }
            }
            if(module.gameObject.GetComponent<BurstTrigger>() != null){
                module.gameObject.GetComponent<BurstTrigger>().OnAttack();
            }
        }
    }
```
# Finding Ray-Plane Intersection
Used in Kaffe3D Engine for clicking objects in edit mode.
``` C++
// Finds the intersection between a line segment and a plane.
// @param[in] p0 Starting point of line segment.
// @param[in] p1 End point of line segment.
// @param[in] normal Normal vector of the plane.
// @param[in] planePoint any point on the plane.
std::tuple<bool, glm::vec3> Physics::LineSegIntersectPlane(glm::vec3 p0, glm::vec3 p1, glm::vec3 normal, glm::vec3 planePoint)
{
	glm::vec3 lineDir = glm::normalize(p1-p0);
	float dotLineNorm = glm::dot(normal, lineDir);

	// Only care about planes facing the ray (Determined by negative dot product).
	if (dotLineNorm < -1e-6)
	{
		// We solve for the collision point by putting the ray in parametric form (StartingPoint + Direction * t = intersection)
		// A line segment on the plane can formed from two points on the plane. The dot product of this line and the plane normal is zero since they are parallel.
		// One point on the plane will be the intersection, which we can substitute with our parametric form ray.
		// We solve for t then plug t back into the ray equation.
		
		// After algebra
		float t = glm::dot(planePoint - p0, normal) / glm::dot(lineDir, normal);

		if(t > 1e-6)
			return std::tuple<bool,glm::vec3>(true, p0 + (t * lineDir));
		else
			return std::tuple<bool, glm::vec3>(false, glm::vec3(0));
	}
	else
		return std::tuple<bool, glm::vec3>(false, glm::vec3(0));
}

```
