<?php

class Listee_CP {
	var $version = '1.0';

	function listee_module_install (){
		global $DB;

		$sql[] = "INSERT INTO exp_modules(module_id, module_name, module_version, has_cp_backend) VALUES('', 'Listee', '$this->version', 'n')";

		foreach($sql as $query) {
			$DB->query($query);
		}

		return true;
	}

	function listee_module_deinstall() {
		global $DB;

		$query = $DB->query("SELECT module_id FROM exp_modules WHERE module_name = 'Listee'");
		$sql[] = "DELETE FROM exp_module_member_groups WHERE module_id = '".$query->row['module_id']."'";
		$sql[] = "DELETE FROM exp_modules WHERE module_name = 'Listee'";
		$sql[] = "DELETE FROM exp_actions WHERE class = 'Listee'";
		$sql[] = "DELETE FROM exp_actions WHERE class = 'Listee_CP'";
		$sql[] = "DROP TABLE IF EXISTS exp_listee";

		foreach ($sql as $query) {
			$DB->query($query);
		}

		return true;
	}
}

?>