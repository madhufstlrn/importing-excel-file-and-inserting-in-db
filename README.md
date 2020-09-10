# importing-excel-file-and-inserting-in-db

# view:bulkdata_import.php


<html class="no-js" lang="en">

<head>
 <title>How to Import Excel Data into Mysql in Codeigniter</title>
 <link rel="stylesheet" href="<?php echo base_url(); ?>asset/bootstrap.min.css" />
 <script src="<?php echo base_url(); ?>asset/jquery.min.js"></script>
</head>
 <div class="content-body">
<body>
 <div class="container">
  <br />
  <h3 align="center"> Importing Excel Data into TempTable</h3>
  <form method="post" id="import_form" enctype="multipart/form-data">
   <p><label>Select Excel File</label>
   <input type="file" name="file" id="file" required accept=".xls, .xlsx" /></p>
   <br />
   <input type="submit" name="import" value="Import" class="btn btn-info" />
  </form>
  <br />
  <div class="table-responsive" id="customer_data">

  </div>
 </div>
</body>
 </div>

</html>

 <script>
$(document).ready(function(){

// load_data();
//
// function load_data()
// {
//  $.ajax({
//   url:"<?php echo base_url(); ?>Econtractor_controller/fetch",
//   method:"POST",
//   success:function(data){
//    $('#customer_data').html(data);
//   }
//  })
// }

 $('#import_form').on('submit', function(event){
  event.preventDefault();
  $.ajax({
   url:"<?php echo base_url(); ?>Econtractor_controller/import",
   method:"POST",
   data:new FormData(this),
   contentType:false,
   cache:false,
   processData:false,
   success:function(data){
    $('#file').val('');
//    load_data();
    alert(data);
   }
  })
 });

});
</script>
# view end:

# controller:

 
  public function bulkdata_import() {
//        $this->load->library('excel');
        $this->load->view('bulkdata_import');
    }
    
 public function import() {
//        echo "hi";
        $this->load->library('excel');
        if (isset($_FILES["file"]["name"])) {
            $file_name = $_FILES['file']['name'];
            $path = $_FILES["file"]["tmp_name"];
            $objReader = PHPExcel_IOFactory::createReader('Excel2007');

            $worksheetList = $objReader->listWorksheetNames($path);
            $sheetname = $worksheetList[0];
            $objReader->setLoadSheetsOnly($sheetname);

            $worksheetData = $objReader->listWorksheetInfo($path);
            $totalRows = $worksheetData[0]['totalRows'];
            $totalColumns = $worksheetData[0]['totalColumns'];
            $object = $objReader->load($path);
//   $object = PHPExcel_IOFactory::load($path);

            foreach ($object->getWorksheetIterator() as $worksheet) {
                 if($worksheet != NULL ) { 
//    $highestRow = $worksheet->getHighestDataRow();
//    $highestColumn = $worksheet->getHighestColumn();
                for ($row = 3; $row <= $totalRows; $row++) {

                    $contractor_name = $worksheet->getCellByColumnAndRow(0, $row)->getValue();
                    $unit_name = $worksheet->getCellByColumnAndRow(1, $row)->getValue();
                    $empcode = $worksheet->getCellByColumnAndRow(2, $row)->getValue();
        //            for date to get in dd/mm/yyyy format
      //              $increment_date = PHPExcel_Style_NumberFormat::toFormattedString($worksheet->getCellByColumnAndRow(45, $row)->getCalculatedValue(), 'dd/mm/yyyy');
      //                $date_of_resignation = $worksheet->getCellByColumnAndRow(80, $row)->getFormattedValue();
      //             ending for date to get in dd/mm/yyyy format
                      $contractor_namelen = strlen($contractor_name);
                    if (isset($contractor_name)) {
                        if ($contractor_namelen >= 10) {

                            if (ctype_alpha(str_replace(' ', '', $contractor_name)) === false) {
                                echo "contractorname must contain letters and spaces only";
                                return false;
                            }

                            $contractor_name = $contractor_name;
                        } else {
                            echo " contractorname must be at least 10 characters";
                            return false;
                        }
                    } else {
                        echo "contractor name should not be empty.contractorname must be at least 10 characters";
                        return false;
                    }
//              contractor name validation end
//              Unit name validation start
                    if (isset($unit_name)) {
                        if (ctype_alpha(str_replace(' ', '', $unit_name)) === false) {
                            echo "UnitName must contain letters and spaces only";
                            return false;
                        }
                        $unit_name = $unit_name;
                    } else {
                        echo "unitname should not be empty.";
                        return false;
                    }
//                   Unit name validation end
//                    Employee code validation start
                    $empcode_len = strlen($empcode);
                    if ($empcode_len > 8 || $empcode_len < 7) {
                        echo "employee code should be 7 or 8 numbers only.";
                        return false;
                    }
                    if (isset($empcode)) {
                        if (is_numeric($empcode)) {
                            $empcode = $empcode;
                        } else {
                            echo "employee code should be numeric only.";
                            return false;
                        }
                    } else {
                        echo "employee code should not be empty.";
                        return false;
                    }
//                  Employee code validation start



                    $data[] = array(
                        'cont040_contractor_name' => $contractor_name,
                        'cont040_unit_name' => $unit_name,
                        'cont040_empcode' => $empcode
                         );
                }
            }
            }
//   print_r($data);exit;
            $result = $this->Econtractor_model->insert($data);
            if ($result)
                echo "Data Imported successfully";
        }
    }
    
    
   # Model:
    
    function insert($data)
 {
//     print_r($data);
//     echo "hi";
//    if(count($data) > 0){
// 
//      // Check user
//      $this->db->select('*');
//      $this->db->where('cont040_empcode', $empcode);
//      $q = $this->db->get('cont040_temp_emp_info');
//      $response = $q->result_array();
 
      // Insert record
//      if(count($response) == 0){
  $query = $this->db->insert_batch('cont041_temp_emp_info', $data);
  return $query;
//  echo $this->db->last_query();exit;
// }
// }
 }
    
    
