.. _cn_api_fluid_layers_switch_case:

switch_case
-------------------------------


.. py:function:: paddle.static.nn.switch_case(branch_index, branch_fns, default=None, name=None)


运行方式类似于c++的switch/case。

参数
::::::::::::

    - **branch_index** (Tensor)- 形状为[1]的Tensor，指定将要执行的分支。数据类型是 ``int32``, ``int64`` 或 ``uint8``。
    - **branch_fns** (dict|list|tuple) - 如果 ``branch_fns`` 是一个list或tuple，它的元素可以是 (int, callable) 二元组，即由整数和可调用对象构成的二元组，整数表示对应的可调用对象的键；也可以仅仅是可调用对象，它在list或者tuple中的实际索引值将作为该可调用对象的键。如果 ``branch_fns`` 是一个字典，那么它的键是整数，它的值是可调用对象。所有的可调用对象都返回相同结构的Tensor。
    - **default** (callable，可选) - 可调用对象，返回一个或多个张量。
    - **name** (str，可选) – 具体用法请参见 :ref:`api_guide_Name` ，一般无需设置，默认值：None。

返回
::::::::::::

Tensor|list(Tensor)

- 如果 ``branch_fns`` 中存在与 ``branch_index`` 匹配的可调用对象，则返回该可调用对象的返回结果；如果 ``branch_fns`` 中不存在与 ``branch_index`` 匹配的可调用对象且 ``default`` 不是None，则返回调用 ``default`` 的返回结果；
- 如果 ``branch_fns`` 中不存在与 ``branch_index`` 匹配的可调用对象且 ``default`` 是None，则返回 ``branch_fns`` 中键值最大的可调用对象的返回结果。

代码示例
::::::::::::

.. code-block:: python

    import paddle

    paddle.enable_static()

    def fn_1():
        return paddle.full(shape=[1, 2], dtype='float32', fill_value=1)

    def fn_2():
        return paddle.full(shape=[2, 2], dtype='int32', fill_value=2)

    def fn_3():
        return paddle.full(shape=[3], dtype='int32', fill_value=3)

    main_program = paddle.static.default_startup_program()
    startup_program = paddle.static.default_main_program()
    with paddle.static.program_guard(main_program, startup_program):
        index_1 = paddle.full(shape=[1], dtype='int32', fill_value=1)
        index_2 = paddle.full(shape=[1], dtype='int32', fill_value=2)

        out_1 = paddle.static.nn.switch_case(
            branch_index=index_1,
            branch_fns={1: fn_1, 2: fn_2},
            default=fn_3)

        out_2 = paddle.static.nn.switch_case(
            branch_index=index_2,
            branch_fns=[(1, fn_1), (2, fn_2)],
            default=fn_3)

        # Argument default is None and no index matches. fn_3 will be called because of the max index 7.
        out_3 = paddle.static.nn.switch_case(
            branch_index=index_2,
            branch_fns=[(0, fn_1), (4, fn_2), (7, fn_3)])

        exe = paddle.static.Executor(paddle.CPUPlace())
        res_1, res_2, res_3 = exe.run(main_program, fetch_list=[out_1, out_2, out_3])
        print(res_1)  # [[1. 1.]]
        print(res_2)  # [[2 2] [2 2]]
        print(res_3)  # [3 3 3]