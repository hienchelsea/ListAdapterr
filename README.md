# RecyviewAdapter
# DiffUtil
# ListAdapter
# Template ListAdapter

### 1.RecyviewAdapter
##### a)Tổng quan: 
  RecyclerView là một ViewGroup nó được dùng để chuẩn bị và hiện thị các View tương tự nhau. RecyclerView được cho là sự kế thừa của ListView và GridView , và nó được giới thiệu trong phiên bản suport-v7
##### b)Cách triển khai: 
  Kế thừa RecyclerView.Adapter<RecyclerView.ViewHolder>()

    class TaskListAdapter : RecyclerView.Adapter<ViewHolder>() {
  
    private val tasks: MutableList<Task> = ArrayList()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return ViewHolder(inflater.inflate(R.layout.item_task_row, parent, false))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(tasks[position])
    }

    override fun getItemCount(): Int = tasks.size

    fun addTask(task: Task) {
        if (!tasks.contains(task)) {
            tasks.add(task)
            notifyItemInserted(tasks.size)
        }
    }

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        fun bind(task: Task) {
            itemView.taskTitle.text = task.title
        }

    }
}
##### c)Vấn đề: 
  Vấn đề lớn nhất trong đoạn code này là nó chỉ xử lý việc thêm một Task mới và không xử lý việc xóa hoặc sử một Task. Nếu chúng ta muốn dễ dàng xử lý những trường hợp này, chúng ta có thể sử dụng notifyDataSetChanged() nhưng chúng ta sẽ đánh mất những animation đẹp. Để xử lý chúng đúng, chúng ta cần làm nhiều việc hơn một chút đó là sử dụng DIfUtil.
  
  
### 2.DiffUtil
##### a)Tổng quan: 
   Khi nội dung bị thay đổi, chúng ta phải gọi notifyDataSetChanged() để cập nhận dữ liệu mới nhưng nó lại rất tốn công. Cần rất nhiều lần lặp lại để hoàn thành trong việc gọi notifyDataSetChanged.Vì thế DiffUtil class xuất hiện và Android đã phát triển class tiện ích này để xử lý việc cập nhập dữ liệu cho RecyclerView.
  
   Kể từ 24.2.0 , RecyclerView support library, v7 package cung cấp lớp tiện ích có tên là DiffUtil. Class này tìm sự khác nhau giữa 2 lists và cung cấp danh sách mới dưới dạng output. Lớp này được sử dụng để thông báo cấp nhập cho RecyclerView Adapter.
  
   Về cơ bản DiffUtil vẫn sử dụng các method của RecyclerView.Adapter để thông báo cho adapter cập nhật dư liệu như:
   
      notifyItemChange()
      notifyItemMoved()
      notifyItemInserted()
      
  Và sử dụng thuật toán của Eugene W. Myers để tính số cập nhật tối thiểu
##### b)Cách triển khai: 
    getOldListSize()
   – Trả về số lượng của list cũ

    getNewListSize()
   – Trả về số lượng của list mới

    areItemsTheSame(int oldItemPosition, int newItemPosition)
   – Nó quyết định xem 2 đối tượng có cùng Items hay là không

    areContentsTheSame(int oldItemPosition, int newItemPosition)
   – Nó quyết định xem 2 Items có cùng dữ liệu hay là không. Phương thức này chỉ được gợi khi areItemsTheSame() trả về true.

    getChangePayload(int oldItemPosition, int newItemPosition)
   – Nếu areItemTheSame() trả về true và areContentsTheSame() trả false sau đó DiffUtil sẽ gọi phương thức này để trả về sự thay đổi.


    class UserDiffUtil(
    private val mOldUsers: List<User>, 
    private val mNewUsers: List<User>
    ) : DiffUtil.Callback() {
  
    override fun getOldListSize(): Int {
        return mOldUsers.size
    }

    override fun getNewListSize(): Int {
        return mNewUsers.size
    }

    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return mOldUsers[oldItemPosition].id === mNewUsers[newItemPosition].id
    }

    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        val oldUserName = mOldUsers[oldItemPosition].name
        val newUserName = mNewUsers[newItemPosition].name
        return oldUserName == newUserName
    }

    @Nullable
    override fun getChangePayload(oldItemPosition: Int, newItemPosition: Int): Any? {
        return super.getChangePayload(oldItemPosition, newItemPosition)
    }}}

Và triển khai trong Adapter

    fun submitList(users: List<User>) {
    val diffResult = DiffUtil.calculateDiff(UserDiffUtil(mUsers, users))
    mUsers.clear()
    mUsers.addAll(users)
    diffResult.dispatchUpdatesTo(this)}
    
##### c)Vấn đề: 
  Không có vấn đề gì cả, chỉ là chưa tối ưu được hết DiffUtil
  
### 3.ListAdapter
##### a)Tổng quan:  
   Từ những hiệu quả mà DiffUtil mang lại thì trong support library 27.1.0 ListAdapter ra đời nó là 1 Wraper của RecyclerView.Adapter giúp cho đơn giản hóa code cần thiết để làm việc với RecyclerView và có thể tự động lưu trữ list Item cũ và sử dụng DiffUtil để chỉ update những items có sự thay đổi. Điều này cho hiệu năng tốt hơn vì bạn tránh refreshing toàn bộ list, và có những animation tốt bởi vì chỉ những item thay đổi cần vẽ lại.
 
 Đơn giản hóa code hơn, không cần phải viết nhiều code nữa :v
 ##### b)Cách triển khai: 
   Kế thừa ListAdapter<T, Recyview.ViewHolder>(DiffItemCallBack)
 
    class DummyAdapter : ListAdapter<Dummy, RecyclerView.ViewHolder>(DummyDiffUtilCallback()) {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return ViewHolderFile(inflater.inflate(R.layout.item_dummy, parent, false))
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        (holder as ViewHolderFile).bind(currentList[position])
    }

    inner class ViewHolderFile(itemView: View) : RecyclerView.ViewHolder(itemView) {

        fun bind(item: Dummy) {

        }
    }}
    
##### c)DiffUtil.ItemCallback
  DiffUtil.ItemCallback đây là phiên bản rút gọi của callback DiffUtil.

  DiffUtil.ItemCallBack yêu cầu bạn implment 2 function areItemsTheSame và areContentsTheSame

    areItemTheSame
areItemsTheSame đưa cho bạn 2 item và hỏi bạn xem chúng có đại diện cho cùng một đối tượng không. Trong trường hợp của tôi, tôi có ID duy nhất tôi sử dụng nó để so sánh. Nếu IDs phù hợp, sau đó tôi biết những items này giống nhau ngay cả khi có một số trường khác nhau.

    areContentsTheSame
areContenTheSame lại cho bạn 2 item. Lần này, nó hỏi bạn xem 2 item này có cùng nội dung không. Nếu bạn quyết định những item của bạn có nội dung giống nhau, sau đó nó không vẽ lại và không có animation xảy ra. Tuy nhiên, nếu bạn quyết định nội dung không giống nha thì item sẽ được vẽ lại trên màn hình

    class DummyDiffUtilCallback: DiffUtil.ItemCallback<Dummy>() {
    override fun areItemsTheSame(oldItem: Dummy, newItem: Dummy): Boolean {
        return oldItem.id == newItem.id
    }

    override fun areContentsTheSame(oldItem: Dummy, newItem: Dummy): Boolean {
        return oldItem == newItem
    }
    }
    
 ##### d)Payload
    Có cách nào để có được danh sách tất cả ViewHolders để sau đó cập nhật (animate) chỉ một cái đó TextView bên trong mỗi cái không?
Có thể thông báo cho người quan sát của RecyclerView.Adapter Để phát hành bản cập nhật một phần RecyclerView.ViewHolders Bằng cách chuyển tải trọng Object.

    override fun onBindViewHolder(
        holder: RecyclerView.ViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        if (payloads.isEmpty()) {
            super.onBindViewHolder(holder, position, payloads)
        } else {
            if (payloads.any { it is InfoMessageChanged }) {
                (holder as ViewHolderFile).bind(currentList[position])
            }
        }
    }
    
##### e)Callback in Recyview   
  Kottlin khuyết khích sử dụng hàm lamda thay cho interface callback vì ngắn gọn hơn, tường minh hơn.

    val itemClickListener: (String) -> Unit)
    
##### c)Vấn đề: 
  Vẫn thấy viết nhiều code vlone nếu phải code đi code lại hết apdater này sang adapter khác :v

### Template ListAdapter

Tốc biến không qua tường :v

    #if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}

    #end

    import android.view.LayoutInflater
    import android.view.View
    import android.view.ViewGroup
    import androidx.recyclerview.widget.ListAdapter
    import androidx.recyclerview.widget.RecyclerView

    #parse("File Header.java")
    class  ${NAME} : ListAdapter<${MODEL}, RecyclerView.ViewHolder>(${DIFF_UTIL_CLASS}()) {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return ${VIEWHOLDER_CLASS}(inflater.inflate(R.layout.${LAYOUT_RES_ID}, parent, false))
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        (holder as ${VIEWHOLDER_CLASS}).bind(currentList[position])
    }

    override fun onBindViewHolder(
        holder: RecyclerView.ViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        if (payloads.isEmpty()) {
            super.onBindViewHolder(holder, position, payloads)
        } else {
            if (payloads.any { it is ${INFOR_CLASS} }) {
                (holder as ${VIEWHOLDER_CLASS}).bind(currentList[position])
            }
        }
    }

    inner class ${VIEWHOLDER_CLASS}(itemView: View) : RecyclerView.ViewHolder(itemView) {

        fun bind(item: Dummy) {

        }
    }

    class ${INFOR_CLASS}
    }
    
    
END_GAME


    
  
 

  
  

  

