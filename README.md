#

时间复杂度： 一个算法的执行时间

空间复杂度： 一个算法执行过程中临时占用存储空间的大小

1、快速排序：

   public void quitSort(int[] array, int left, int right){
       if(left >= right){
           return;    
       }
       int i = left;
       int j = right;
       int temp = a[left]；
       while(i < j){
           while(a[j] > temp && i < j){
               j--;            //从后往前搜，比temp更小的值就放到前面
           }
           a[i] == a[j]；
           while(a[i] <= temp && i < j){
              i++;    
           }
           array[j] == a[i]
       }
       array[i] = temp;
       quitSort(array, left, i-1);
       quitSort(array, i+1, right);
   }
