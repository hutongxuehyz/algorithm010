学习笔记：
	1.加一
		刚开始我的思路是这样的，将int数组转化为字符串然后再转换成Integer加1，加一之后的数再转换为字符串再转换成char数组，然后遍历将char转换成int放入返回的数组中，
		这样的问题是数组长度比较大的话数组所表示的数会比Integer类型的值要大，程序执行出错，思考之后没有想到更好的思路来解决，就去题解里看了一下高票答案，理解之后就自己动手
		解决了这个问题。
		public int[] plusOne(int[] digits) {
        int[] result = new int[digits.length];
        if(digits.length == 1 && digits[0] == 0){
            result[0] = 1;
            return result;
        }
        String str = "";
        for(int i = 0 ; i < digits.length ; i ++){
            str = str + digits[i];
        }
        Integer sum = Integer.valueOf(str) + 1;//这里会报java.lang.NumberFormatException
        char[] ch = sum.toString().toCharArray();
        if(ch.length > result.length){
            result = new int[ch.length];
        }
        for(int i = 0 ; i < result.length ; i ++){
            result[i] = Integer.valueOf(String.valueOf(ch[i]));
        }
        return result;
    }
	
	2.旋转数组
		刚开始写代码时，没有考虑到要移动的长度K会比数组的长度大，所以第一次提交代码时报错了
			int[] temporary = new int[nums.length];
            for(int i = 0 ; i < nums.length ; i ++){
            if(i < k){
                temporary[i] = nums[nums.length - k + i];//java.lang.ArrayIndexOutOfBoundsException: Index -1 out of bounds for length
            }else{
                temporary[i] = nums[i - k];
            }
            }
            for(int i = 0 ; i < nums.length ; i ++){
             nums[i] = temporary[i];
            }
		经过修改之后，执行程序前我对k和数组长度做了一次比较，如果k大于数组长度时，k对数组长度取余为新的k
		